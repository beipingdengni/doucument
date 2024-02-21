## RuleBook

RuleBook提供了一个简单、灵活并且直观的DSL。RuleBook提供了易于使用的Lambda特定语言或POJO方式来定义规则，Java开发人员可以通过带注解的POJO来组织大规模规则集合，替代那些又臭又长的“if/else”

#### Maven

```xml
<dependency>
    <groupId>com.deliveredtechnologies</groupId>
    <artifactId>rulebook-core</artifactId>
    <version>${version}</version>
</dependency>
```

#### Java 定义规则

```java
public class Cart{
    private double cartTotal;
    private String cartId;
    private Customer user;
    private List cartEntries;
    
    //getter and setter
}

// 定制规则
public class ShoppingCartRule extends CoRRuleBook {

    @Override
    public void defineRules() {

    //give 10% off when cart total is greater than $200
    addRule(RuleBuilder.create().withFactType(Cart.class).withResultType(Double.class)
            .when(facts -> facts.getOne().getCartTotal() > 200)
            .then((facts, result) -> result.setValue(20))
            .stop()
            .build());
}

// 执行
public class CartPromotionRule {
    public static void main(String[] args) {
      RuleBook cartPromotion = RuleBookBuilder.create(ShoppingCartRule.class).withResultType(Double.class)
        .withDefaultResult(0.0)
        .build();
      NameValueReferableMap facts = new FactMap();
      facts.setValue("cart", new Cart(450.0, 123456, customer, entries));
      cartPromotion.run(facts);
    }
}
```

