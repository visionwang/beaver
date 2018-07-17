https://www.cnblogs.com/woshimrf/p/6637686.html?utm_source=itdadao&utm_medium=referral

/**
 * Bean Validation 中内置的 constraint       
 * @Null   被注释的元素必须为 null       
 * @NotNull    被注释的元素必须不为 null       
 * @AssertTrue     被注释的元素必须为 true       
 * @AssertFalse    被注释的元素必须为 false       
 * @Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值       
 * @Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值       
 * @DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值       
 * @DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值       
 * @Size(max=, min=)   被注释的元素的大小必须在指定的范围内       
 * @Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内       
 * @Past   被注释的元素必须是一个过去的日期       
 * @Future     被注释的元素必须是一个将来的日期       
 * @Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式       
 * Hibernate Validator 附加的 constraint       
 * @NotBlank(message =)   验证字符串非null，且长度必须大于0       
 * @Email  被注释的元素必须是电子邮箱地址       
 * @Length(min=,max=)  被注释的字符串的大小必须在指定的范围内       
 * @NotEmpty   被注释的字符串的必须非空       
 * @Range(min=,max=,message=)  被注释的元素必须在合适的范围内 
 */



 @ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(Exception.class)
public CodeMsg ex(MethodArgumentNotValidException exception, HttpServletResponse response){
    LOGGER.error("请求参数不合法。", exception);
    BindingResult bindingResult = exception.getBindingResult();
    String msg = "校验失败";
    return new CodeMsg(CodeMsgConstant.error, msg, getErrors(bindingResult));
}

private Map<String, String> getErrors(BindingResult result) {
    Map<String, String> map = new HashMap<>();
    List<FieldError> list = result.getFieldErrors();
    for (FieldError error : list) {
        map.put(error.getField(), error.getDefaultMessage());
    }
    return map;
}

public class AlipayRequest {
@NotEmpty
private String out_trade_no;
private String subject;
@DecimalMin(value = "0.01", message = "费用最少不能小于0.01")
@DecimalMax(value = "100000000.00", message = "费用最大不能超过100000000")
private String total_fee;
/**
 * 订单类型
 */
@NotEmpty(message = "订单类型不能为空")
private String business_type;

//....
}

@RequestMapping(value = "sign", method = RequestMethod.POST)
    public String sign(@Valid @RequestBody AlipayRequest params
    ){
        ....
    }

    <dependency>  
   <groupId>org.hibernate</groupId>  
   <artifactId>hibernate-validator</artifactId>  
   <version>5.0.2.Final</version>  
</dependency>


<mvc:annotation-driven validator="validator" />
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
  <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
  <property name="validationMessageSource" ref="messageSource"/>
</bean>


904