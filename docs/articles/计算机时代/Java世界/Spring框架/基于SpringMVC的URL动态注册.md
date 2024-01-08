# #基于Spring MVC的URL动态注册

## 如何动态注册URL

利用SpringMVC提供的`RequestMappingHandlerMapping`​类实现动态绑定与解绑.

```java
@RestController
@RequestMapping("/dynamic")
public class DynamicUrlController {

    @Autowired
    private RequestMappingHandlerMapping requestMappingHandlerMapping;
    @Autowired
    private DynamicUrlHandlerController  dynamicUrlHandlerController;

    @PostMapping("/register")
    public RespDTO register(@RequestBody DynamicParamVO dynamicParam) throws NoSuchMethodException {
        String url = "/api/" + dynamicParam.getUrl();
        RequestMappingInfo requestMappingInfo = RequestMappingInfo.paths(url).methods(RequestMethod.GET, RequestMethod.POST).build();
        Method method = dynamicUrlHandlerController.getClass().getDeclaredMethod("doDynamicUrl", HttpServletRequest.class);
        requestMappingHandlerMapping.registerMapping(requestMappingInfo, dynamicUrlHandlerController, method);
        RequestParamHelper.getINSTANCE().putParam(url, dynamicParam.getParams());
        return RespDTO.ok();
    }


    @GetMapping("/list")
    public RespDTO registerList() {
        List<String> urls = new ArrayList();
        Map<RequestMappingInfo, HandlerMethod> handlerMethods = requestMappingHandlerMapping.getHandlerMethods();
        for (RequestMappingInfo requestMappingInfo : handlerMethods.keySet()) {
            urls.add(requestMappingInfo.toString());
        }
        return RespDTO.ok(urls);
    }
}
```

### 示例动态程序

```java
@RestController
@RequestMapping("/api")
public class DynamicUrlHandlerController {

    @Autowired
    private HikariDataSource    dataSource;
    @Autowired
    private SqlConfigRepository sqlConfigRepository;

    public RespDTO doDynamicUrl(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        RequestWrapper requestWrapper = new RequestWrapper(request);
        RequestParamHelper instance = RequestParamHelper.getINSTANCE();
        Map<String, Object> params = instance.extraGetParams(requestWrapper);
        params = instance.extraBodyParams(requestWrapper.getBodyString(), params);
//        String sql = "update t_user set email = \"test@qq.com\" where id = 5";
        SqlConfigEntity querySqlConfigEntity = new SqlConfigEntity();
        querySqlConfigEntity.setUrl(requestURI);
        List<SqlConfigEntity> configEntities = sqlConfigRepository.findAll(Example.of(querySqlConfigEntity));
        if (CollectionUtils.isEmpty(configEntities)) {
            return RespDTO.ok();
        }
        SqlConfigEntity sqlConfigEntity = configEntities.get(0);
        String sql = sqlConfigEntity.getConfigSql();
        try {
            Connection connection = dataSource.getConnection();
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            boolean hasResultSet = preparedStatement.execute();
            if (hasResultSet) {
                ResultSet resultSet = preparedStatement.getResultSet();
                ResultSetMetaData metaData = resultSet.getMetaData();
                //
                List<String> columns = new ArrayList<>();
                int columnCount = metaData.getColumnCount();
                for (int i = 1; i <= columnCount; i++) {
                    columns.add(metaData.getColumnLabel(i));
                }
                //
                List<Map<String, Object>> contents = new ArrayList<>();
                while (resultSet.next()) {
                    Map<String, Object> finalItem = new HashMap<>();
                    columns.stream().forEach(k -> {
                        try {
                            finalItem.put(k, resultSet.getObject(k));
                        } catch (SQLException e) {
                            e.printStackTrace();
                        }
                    });
                    contents.add(finalItem);
                }
                return RespDTO.ok(contents);
            } else {
                // update or delete
                return RespDTO.ok(preparedStatement.getUpdateCount() + " rows affected");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return RespDTO.ok();
    }
}
```
