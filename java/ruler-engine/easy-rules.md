# Easy Rules

### maven

> uRule

```xml
<dependency>
  <groupId>org.jeasy</groupId>
  <artifactId>easy-rules-core</artifactId>
  <version>4.1.0</version>
</dependency>
<dependency>
  <groupId>org.jeasy</groupId>
  <artifactId>easy-rules-support</artifactId>
  <version>4.1.0</version>
</dependency>
```

> 以下是EasyRule的一些核心特性：
>
> - 轻量级类库和容易上手
> - 基于POJO的开发与注解的编程模型
> - 基于MVEL表达式的编程模型（适用于极简单的规则，一般不推荐）
> - 支持根据简单的规则创建组合规则
> - 方便且适用于java的抽象的业务模型规则

### 方式一：注解

```java
@Rule(name = "cart total rule", description = "Give 10% off when shopping cart is greater than $200" )
public class CartTotalRule {

    @Condition
    public boolean cartTotal(@Fact("cart") Cart cart) {
        return cart.isGreaterThanTwoHundered;
    }

    @Action
    public void giveDiscount(@Fact("cart") Cart cart) {
       cart.setTotalDiscount(200);
    }
}
```

### 方式二：链式编程

```java
Rule weatherRule = new RuleBuilder()
    .name("weather rule")
    .description("if it rains then take an umbrella")
    .when(facts -> facts.get("rain").equals(true))
    .then(facts -> System.out.println("It rains, take an umbrella!"))
    .build();
```

### 方式三：表达式

```java
Rule weatherRule = new MVELRule()
    .name("weather rule")
    .description("if it rains then take an umbrella")
    .when("rain == true")
    .then("System.out.println(\"It rains, take an umbrella!\");");
```

### 方式四：yml配置文件

```yaml
name: "weather rule"
description: "if it rains then take an umbrella"
condition: "rain == true"
actions:
 - "System.out.println(\"It rains, take an umbrella!\");"
```

## 执行结果

```java
// 执行
public static void main(String[] args) {
  // define facts
  Facts facts = new Facts();
  facts.put("cart", get_customer_cart);

  // define rules
  Rule cartTotalRUle = CartTotalRule
  Rules rules = new Rules();
  rules.register(cartTotalRUle);

  // fire rules on known facts
  RulesEngine rulesEngine = new DefaultRulesEngine();
	// rulesEngine.registerRuleListener(myRuleListener); //注册监听
  rulesEngine.fire(rules, facts);
}
```

#### 可以通过RuleListener API来监听规则执行事件

```java
public interface RuleListener {

    /**
     * 在评估规则之前触发。
     *
     * @param rule 正在被评估的规则
     * @param facts 评估规则之前的已知事实
     * @return 如果规则应该评估，则返回true，否则返回false
     */
    default boolean beforeEvaluate(Rule rule, Facts facts) {
        return true;
    }

    /**
     * 在评估规则之后触发
     *
     * @param rule 评估之后的规则
     * @param facts 评估规则之后的已知事实
     * @param evaluationResult 评估结果
     */
    default void afterEvaluate(Rule rule, Facts facts, boolean evaluationResult) { }

    /**
     * 运行时异常导致条件评估错误时触发
     *
     * @param rule 评估之后的规则
     * @param facts 评估时的已知事实
     * @param exception 条件评估时发生的异常
     */
    default void onEvaluationError(Rule rule, Facts facts, Exception exception) { }

    /**
     * 在规则操作执行之前触发。
     *
     * @param rule 当前的规则
     * @param facts 执行规则操作时的已知事实
     */
    default void beforeExecute(Rule rule, Facts facts) { }

    /**
     * 在规则操作成功执行之后触发
     *
     * @param rule t当前的规则
     * @param facts 执行规则操作时的已知事实
     */
    default void onSuccess(Rule rule, Facts facts) { }

    /**
     * 在规则操作执行失败时触发
     *
     * @param rule 当前的规则
     * @param facts 执行规则操作时的已知事实
     * @param exception 执行规则操作时发生的异常
     */
    default void onFailure(Rule rule, Facts facts, Exception exception) { }

}
```

