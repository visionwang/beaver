 in
        <foreach collection="nameList" item="name" open="(" separator="," close=")">
            #{name}
        </foreach>
    

http://www.cnblogs.com/fsjohnhuang/p/4078659.html


二、 insert元素 属性详解　　　　　　　　　　　　　　　　　　　　　　　　　　　　

  其属性如下：

   parameterType ，入参的全限定类名或类型别名

   keyColumn ，设置数据表自动生成的主键名。对特定数据库（如PostgreSQL），若自动生成的主键不是第一个字段则必须设置

   keyProperty ，默认值unset，用于设置getGeneratedKeys方法或selectKey子元素返回值将赋值到领域模型的哪个属性中

   useGeneratedKeys ，取值范围true|false(默认值)，设置是否使用JDBC的getGenereatedKeys方法获取主键并赋值到keyProperty设置的领域模型属性中。MySQL和SQLServer执行auto-generated key field，因此当数据库设置好自增长主键后，可通过JDBC的getGeneratedKeys方法获取。但像Oralce等不支持auto-generated key field的数据库就不能用这种方法获取主键了

   statementType ，取值范围STATEMENT,PREPARED（默认值）,CALLABLE

   flushCache ，取值范围true(默认值)|false，设置执行该操作后是否会清空二级缓存和本地缓存

   timeout ，默认为unset（依赖jdbc驱动器的设置），设置执行该操作的最大时限，超时将抛异常

   databaseId ，取值范围oracle|mysql等，表示数据库厂家，元素内部可通过`<if test="_databaseId = 'oracle'">`来为特定数据库指定不同的sql语句

   一定要更新mybatis版本

<trim prefix="where" prefixOverrides="and | or">
           <if test="userStatusList != null and userStatusList.size() > 0">
               UserStatus IN
               <foreach collection="userStatusList" open="(" close=")" separator="," item="userStatus">
                   #{userStatus}
               </foreach>
           </if>
           <if test="stockStatus != null">
               and StockStatus = #{stockStatus}
           </if>
           <if test="payStatus != null">
               and ShopPayStatus = #{payStatus}
           </if>
       </trim>

