# 注释规范 (Comment Standards)

## 适用范围

生成任意代码前必读（代码注释）。规范 Javadoc 与行内注释的覆盖范围、格式与写法。面向所有语言：**通用原则适用任何语言**，示例以 Java（Javadoc）为主；Go/Python/JS 等对照语言习惯套用。

## 强制规则

### 1. 覆盖范围

以下必须写文档注释（Java=Javadoc，其他语言对应 doc 注释）：
- 公开类 / 接口
- 公开方法（含参数、返回值、异常说明）
- 公开字段（尤其含义不直观的）
- 枚举值
- 常量（语义不直观时）

以下不强制 / 禁止：
- private 方法（逻辑复杂时可写）
- **getter/setter、简单转换方法、一行工具方法：禁止加注释**
- 简单重写方法（`@Override` 已有接口注释）

### 2. 类注释格式（Java 示例）

```java
/**
 * 订单查询服务。
 *
 * <p>提供订单的查询、分页、状态流转查询能力。</p>
 *
 * @author zhangsan
 * @since 1.0.0
 */
public class OrderQueryService {
```

- 第一句为概述句，以句号结尾
- `<p>` 分段
- `@since` 记版本；`@author` 团队有要求才写，否则省略

### 3. 方法注释格式

**@param/@return 必须写明业务含义或约束条件，禁止只重复参数名**（如 `@param order 订单` 不达标）。

```java
/**
 * 分页查询订单列表。
 *
 * @param userId    用户 ID，不可为空
 * @param status    订单状态，可空（空表示全部）
 * @param pageNum   页码，从 1 开始
 * @param pageSize  每页条数，最大 100
 * @return 订单分页结果，不含已删除订单
 * @throws BusinessException 用户不存在时抛出
 */
PageResult<OrderVO> queryPage(Long userId, String status, int pageNum, int pageSize);
```

- 参数逐个说明，空值约束写清
- 返回值说明业务含义，非类型复述（写「订单分页结果」，不写「PageResult 对象」）
- 异常条件写清

✅ 正例（写明业务含义 + 约束）：

```java
/**
 * 计算订单实付金额。
 * @param order 订单，含商品明细与优惠信息，不得为 null
 * @return 实付金额，单位分；整单免单时返回 0
 * @throws IllegalArgumentException order.status 非 PAID 状态
 */
```

❌ 反例（只重复参数名，无业务含义）：

```java
/**
 * @param order 订单
 * @return 金额
 */
```

### 4. 行内注释：解释「为什么」，不解释「是什么」

- 复杂逻辑写 `// WHY:` 注释解释"为什么这么做"，禁止写"做了什么"
- 反例：`int total = price * count; // 价格乘以数量`（翻译代码式，禁止）
- 反例：`int i = 0; // 将 i 设为 0`
- 正例：`// WHY: 先扣库存再创建订单，避免超卖窗口`
- 正例：`// 状态为待提交时不允许删除，防止已产生支付记录的订单被误删`
- 魔法值必须注释或提取常量

### 5. 禁止事项

- ❌ 注释掉的代码（用 git 历史管理）
- ❌ 一整块代码全行注释（改为提取方法）
- ❌ 无意义注释：`// TODO 优化`（无具体事项）、`// 处理中` 等
- ❌ 与代码不一致的过时注释（改代码必改注释）
- ❌ 翻译代码式注释（`// 价格乘以数量`）
- ❌ 描述代码中不存在的行为（注释必须与代码事实一致）
- ❌ 拿不准注释是否有价值仍写 —— **宁缺毋滥，不写**

## 反例 / 正例

```java
// 反例
public List<Order> getOrders(Long id) {  // 获取订单
    List<Order> list = orderMapper.selectList(  // 查询列表
        new LambdaQueryWrapper<Order>().eq(Order::getUserId, id));
    return list;
}

// 正例
/**
 * 查询用户的全部订单。
 *
 * @param userId 用户 ID
 * @return 该用户的订单列表，按创建时间倒序
 */
public List<Order> getOrders(Long userId) {
    // 仅查未删除订单，软删除数据不返回
    return orderMapper.selectList(new LambdaQueryWrapper<Order>()
            .eq(Order::getUserId, userId)
            .eq(Order::getDeleted, 0)
            .orderByDesc(Order::getCreateTime));
}
```

## 最佳实践

- 注释解释意图与约束，代码本身表达实现
- 复杂算法（如分库分表路由、状态机）必须注释设计思路
- 特殊业务规则（如「金额单位为分」「时间存 UTC」）注释在字段/常量处
- 团队约定写入统一文档注释头模板（版权、作者、日期按团队要求）

## 自检清单（含验收核对 6 项）

- [ ] 公开 API 注释齐全（每个 public 类/方法有文档注释）
- [ ] @param/@return 有业务含义，不重复参数名，写明约束（如"不得为 null"）
- [ ] 无翻译代码式注释（无 `// 价格乘以数量` 类注释）
- [ ] 复杂逻辑有 `// WHY:`（非显而易见逻辑有"为什么"注释）
- [ ] getter/setter 无注释（简单方法零注释）
- [ ] 注释与代码一致（抽查：注释描述的行为代码真实存在，无编造意图）
- [ ] 无注释掉的代码
- [ ] 无 TODO 无实义注释
