
1.2.4 WithMobileRequestFilter 参数说明
authenticationMode ： 认证模式，可选参数，表示要以何种模式进行认证，支持的模式如下：
  H5Only，只对H5网关来的请求（外网H5请求）验证，不对不经过H5网关的内网的请求进行认证。（默认值）
  OnDemand, 只要head里auth不为空，就验证，（如果认证失败，不进行阻拦，也不会抛出异常）
  Always，总是验证
  ByPass，不认证，如果head里有auth，会原样传回给前端
  BanH5Request，用于只用于内网、不暴露给外网（H5）的操作，当外网（通过H5网关）请求这个操作时，会得到一个403 forbidden的响应。允许内网来的请求，但不会进行认证
  H5Only_AllowNonMemberAuth，但支持非会员认证
  Always_AllowNonMemberAuth，类似于 Always，但支持非会员认证
  H5Only_UseSecondAuth，类似于 H5Only，但使用 head 里的 sauth 进行认证
  Always_UseSecondAuth，类似于 Always，但使用 head 里的 sauth 进行认证
  OnDemand_UseSecondAuth，类似于 OnDemand，但使用 head 里的 sauth 进行认证
  OnDemand_AllowNonMemberAuth，类似于OnDemand，但是支持非会员认证


