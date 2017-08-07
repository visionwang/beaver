1.比较关注的batchInsert是如何实现的？
	//1.方法
	public int combinedInsert(DalHints hints, KeyHolder keyHolder, List<QcAuthorization> daoPojos) throws SQLException {}
	//2.DalRequestExecutor进行处理
	executor.execute(setSize(hints, keyHolder, daoPojos), new DalBulkTaskRequest<>(logicDbName, rawTableName, hints, daoPojos, combinedInsertTask));

	//DalRequest, DalRequestExecutor
	//3.DalRequestExecutor进行处理，目测是除了的核心类
	//为什么使用AtomicReference,可以解决什么问题？
	private static AtomicReference<ExecutorService> serviceRef = new AtomicReference<>();
	public static final int DEFAULT_MAX_POOL_SIZE = 50;
	serviceRef.set(new ThreadPoolExecutor(5, maxPoolSize, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>()));

	public <T> T execute(final DalHints hints, final DalRequest<T> request, final boolean nullable) throws SQLException {
		// TODO add performance tracking DalWatcher.begin();
		//异步执行
		if (hints.isAsyncExecution()) {
			Future<T> future = serviceRef.get().submit(new Callable<T>() {
				public T call() throws Exception {
					return internalExecute(hints, request, nullable);
				}
			});
			
			if(hints.isAsyncExecution())
				hints.set(DalHintEnum.futureResult, future); 
			return null;
		}
		//同步执行
		return internalExecute(hints, request, nullable);
	}
	//hint中可以放入callback
	private <T> T internalExecute(DalHints hints, DalRequest<T> request, boolean nullable) throws SQLException {
		T result = null;
		Throwable error = null;
		
		try {
			request.validate();
			
			/**
			 * TODO make sure detect distributed transaction 
			 */
			//是否跨分片
			if(request.isCrossShard())
				result = crossShardExecute(hints, request);
			else
				result = nonCrossShardExecute(hints, request);

			if(result == null && !nullable)
				throw new DalException(ErrorCode.AssertNull);
		} catch (Throwable e) {
			error = e;
		}
		
		handleCallback(hints, result, error);
		if(error != null)
			throw DalException.wrap(error);
		
		return result;
	}
	//5.关于分片，还需要好好加强
	//异步执行调用（一个Request维度进行），分片后并行执行(如果这个Request涉及到多个表)
	//类似Fork and Join
	/*
	 * To execute CURD in sequential way. 
	 */
	sequentialExecution
	//这儿需要理解清楚，这儿是跨分片的情况下，到时是选择顺序执行还是并行执行
	
	private <T> T crossShardExecute(DalHints hints, DalRequest<T> request) throws SQLException {
		DalWatcher.crossShardBegin();
		
		T result = hints.is(DalHintEnum.sequentialExecution)?
				seqncialExecute(hints, request)://for，之后merger各部分的结果？
				parallelExecute(hints, request);
		
		DalWatcher.crossShardEnd();
		return result;
			
	}
	//进一步异步执行，但还是使用的那个线程池
	private <T> T parallelExecute(DalHints hints, DalRequest<T> request) throws SQLException {
		Map<String, Callable<T>> tasks = request.createTasks();
		Map<String, Future<T>> resultFutures = new HashMap<>();
		
		for(final String shard: tasks.keySet())
			resultFutures.put(shard, serviceRef.get().submit(tasks.get(shard)));

		// TODO Handle timeout and execution exception
		ResultMerger<T> merger = request.getMerger();
		for(Map.Entry<String, Future<T>> entry: resultFutures.entrySet()) {
			try {
				merger.addPartial(entry.getKey(), entry.getValue().get());
			} catch (Throwable e) {
				hints.handleError("There is error during parallel execution: ", e);
			}
		}
		
		return merger.merge();
	}

	//需要一个具体的示例来探究
	
	//6.DalWatch都做的很漂亮，用到了ThreadLocal
	public class DalWatcher {
	private static ThreadLocal<CostRecorder> costRecorder = new ThreadLocal<CostRecorder>();
	}




完整流程
ConsumeRecordDao(单例)

	public ConsumeRecordDao() throws SQLException {
		//解析配置？比如那张表？表的metaInfo
		this.client = new DalTableDao<>(new DalDefaultJpaParser<>(ConsumeRecord.class));
	}


	public final class DalTableDao<T> extends TaskAdapter<T> {
		public static final String GENERATED_KEY = "GENERATED_KEY";
	
		private SingleTask<T> singleInsertTask;
		private SingleTask<T> singleDeleteTask;
		private SingleTask<T> singleUpdateTask;
	
		private BulkTask<Integer, T> combinedInsertTask;
	
		private BulkTask<int[], T> batchInsertTask;
		private BulkTask<int[], T> batchDeleteTask;
		private BulkTask<int[], T> batchUpdateTask;
		
		private DeleteSqlTask<T> deleteSqlTask;
		private UpdateSqlTask<T> updateSqlTask;
	
		private DalRequestExecutor executor; 
		//这个是new出来的?
		public DalTableDao(DalParser<T> parser, DalTaskFactory factory) {
		this(parser, factory, new DalRequestExecutor());
		}
		public DalTableDao(DalParser<T> parser, DalTaskFactory factory, DalRequestExecutor executor) {
			initialize(parser);//表的信息
			initTasks(factory);//CURD操作
			this.executor = executor;//任务执行者
		}
	}

	//TaskAdapter<T> {}
	public static DalClient getClient(String logicDbName) {
        if (logicDbName == null)
            throw new NullPointerException("Database Set name can not be null");

        DalConfigure config = getDalConfigure();

        // Verify if it is existed
        config.getDatabaseSet(logicDbName);

        return new DalDirectClient(config, logicDbName);
	}

	/**
	 * Execute list of commands in the same transaction. This is useful when you have several
	 * commands and you want to combine them in a flexible way.
	 * 
	 * @param commands Container that holds commands
	 * @param hints Additional parameters that instruct how DAL Client perform database operation.
	 * @throws SQLException when things going wrong during the execution
	 */
	void execute(List<DalCommand> commands, DalHints hints) throws SQLException;
	
	//终于找到事务处理的语句了
	public class DalDirectClient implements DalClient {
	
			private <T> T doInTransaction(ConnectionAction<T> action, DalHints hints)
			throws SQLException {
			return transManager.doInTransaction(action, hints);
		}
	
	}

	public <T> T doInTransaction(ConnectionAction<T> action, DalHints hints)throws SQLException{
		action.initLogEntry(connManager.getLogicDbName(), hints);
		action.start();

		Throwable ex = null;
		T result = null;
		int level;
		try {
			level = startTransaction(hints, action.operation);
			action.populateDbMeta();

			result = action.execute();

			endTransaction(level);
		} catch (Throwable e) {
			ex = e;
			rollbackTransaction();
			MarkdownManager.detect(action.connHolder, action.start, e);
		}finally{
			action.cleanup();
		}

		action.end(result, ex);

		return result;
	}


	//DatabaseSet拿元数据,从DalConfigure
	//分片,Client
	public static DalConfigure load() throws Exception {
	    URL dalconfigUrl = getDalConfigUrl();
	    if (dalconfigUrl == null)
	        throw new IllegalStateException(
	                "Can not find " + DAL_XML + " or " + DAL_CONFIG + " to initilize dal configure");
	
	    return load(dalconfigUrl);
	}


	
	//getFromDocument，想到了zk的树
        Map<String, DatabaseSet> databaseSets = readDatabaseSets(getChildNode(root, DATABASE_SETS));


	private DatabaseSet readDatabaseSet(Node databaseSetNode) throws Exception {
	    List<Node> databaseList = getChildNodes(databaseSetNode, ADD);
	    Map<String, DataBase> databases = new HashMap<>();
	    for (int i = 0; i < databaseList.size(); i++) {
	        DataBase database = readDataBase(databaseList.get(i));
	        databases.put(database.getName(), database);
	    }
	
	    if (hasAttribute(databaseSetNode, SHARD_STRATEGY))
	        return new DatabaseSet(getAttribute(databaseSetNode, NAME), getAttribute(databaseSetNode, PROVIDER),
	                getAttribute(databaseSetNode, SHARD_STRATEGY), databases);
	    else if (hasAttribute(databaseSetNode, SHARDING_STRATEGY))
	        return new DatabaseSet(getAttribute(databaseSetNode, NAME), getAttribute(databaseSetNode, PROVIDER),
	                getAttribute(databaseSetNode, SHARDING_STRATEGY), databases);
	    else
	        return new DatabaseSet(getAttribute(databaseSetNode, NAME), getAttribute(databaseSetNode, PROVIDER),
	                databases);
	}

	private void initStrategy(String shardStrategy) throws Exception {
		if(shardStrategy == null || shardStrategy.length() == 0)
			return;
		
		String[] values = shardStrategy.split(ENTRY_SEPARATOR);
		String[] strategyDef = values[0].split(KEY_VALUE_SEPARATOR);
		
		if(strategyDef[0].trim().equals(CLASS))
			strategy = (DalShardingStrategy)Class.forName(strategyDef[1].trim()).newInstance();
		Map<String, String> settings = new HashMap<String, String>();
		for(int i = 1; i < values.length; i++) {
			String[] entry = values[i].split(KEY_VALUE_SEPARATOR);
			settings.put(entry[0].trim(), entry[1].trim());
		}
		strategy.initialize(settings);
	}

	public void contextInitialized(ServletContextEvent sce) {
			logger.info("Dal Factory Listener is about to start");
			ServletContext context = sce.getServletContext();
			
			String DalConfigPath = context.getInitParameter("com.ctrip.platform.dal.dao.DalConfigPath");
			String warmUp = context.getInitParameter("com.ctrip.platform.dal.dao.DalWarmUp");
	
			try {
				if(DalConfigPath == null || DalConfigPath.trim().length() == 0)
					DalClientFactory.initClientFactory();
				else
					DalClientFactory.initClientFactory(DalConfigPath.trim());
				
				if(Boolean.parseBoolean(warmUp))
					DalClientFactory.warmUpConnections();
	
			} catch (Throwable e) {
				logger.error("Error when init client factory", e);
				throw new RuntimeException(e);
			}
		}

	public class DalClientFactory {
		private static Logger logger = LoggerFactory.getLogger(Version.getLoggerName());
		private static AtomicReference<DalConfigure> configureRef = new AtomicReference<DalConfigure>();
	
		private static void internalInitClientFactory(String path) throws Exception {
		        if (configureRef.get() != null) {
		            logger.warn("Dal Java Client Factory is already initialized.");
		            return;
		        }
		
		        synchronized (DalClientFactory.class) {
		            if (configureRef.get() != null) {
		                return;
		            }
		
		            DalConfigure config = null;
		            if (path == null) {
		                DalConfigLoader loader = ServiceLoaderHelper.getInstance(DalConfigLoader.class);
		                if (loader == null)
		                    config = DalConfigureFactory.load();
		                else
		                    config = loader.load();
		                logger.info("Successfully initialized Dal Java Client Factory");
		            } else {
		                config = DalConfigureFactory.load(path);
		                logger.info("Successfully initialized Dal Java Client Factory with " + path);
		            }
		
		            DalWatcher.init();
		            DalRequestExecutor.init(config.getFacory().getProperty(DalRequestExecutor.MAX_POOL_SIZE));
		            DalStatusManager.initialize(config);
		
		            configureRef.set(config);
		        }
		    }
	}
	//批量事务
	@Override
	public void execute(final List<DalCommand> commands, final DalHints hints)
			throws SQLException {
		final DalClient client = this;
		ConnectionAction<?> action = new ConnectionAction<Object>() {
			@Override
			public Object execute() throws Exception {
				for(DalCommand cmd: commands) {
					if(!cmd.execute(client))
						break;
				}
				
				return null;
			}
		};
		action.populate(commands);
		
		doInTransaction(action, hints);
	}

其实就是单个连接，单个DB？
这是DAL组件

那么想想Mybatis是如何实现的？


问题，分片算法是如何实现的？Dal

Mybatis和Spring是如何继承Transactional?