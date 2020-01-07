> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 http://westerly-lzh.github.io/cn/2014/12/JPA-CascadeType-Explaining/

## Background

网上关于 JPA 的 CascadeType 讲解很多，但几乎都说的很模糊. 本文试图使用一个具体的例子来说明 CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REFRESH, CascadeType.REMOVE, CascadeType.ALL 具体区别。

首先，我们使用一个订单和订单项的例子。该例子在网络上那些介绍 JPA CascadeType 用法的文章钟广为流传。

```
/**
 * 订单
 */
@Entity
@Table()
public class Order {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	@Column
	private String name;
	@OneToMany(mappedBy="order",targetEntity=Item.class,fetch=FetchType.LAZY)
	private List<Item> items;
｝

/**
 *订单项
 */
@Entity
@Table()
public class Item {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	@Column
	private String name;
	@ManyToOne(fetch=FetchType.LAZY,targetEntity=Order.class)
	@JoinColumn()
	private Order order;
｝

/** Order Repository */
public interface OrderRepository extends JpaRepository<Order, Integer>,
	JpaSpecificationExecutor<Order> {

}

/** Item Repository */
public interface ItemRepository extends JpaRepository<Item, Integer>,
	JpaSpecificationExecutor<Item> {

}

```

## 场景 1，新增(保存) 数据 (`CascadeType.PERSIST`)

客户每次下完订单后，需要保存 Order，但是订单里含有 Item，因此，在保存 Order 时候，Order 相关联的 Item 也需要保存。采用上面的模型，使用如下的测试代码：

```
@Test
public void addTest(){
	Order order = new Order();
	order.setName("order1");

	Item item1 = new Item();
	item1.setName("item1_order1");
	item1.setOrder(order);

	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);

	List<Item> items = new ArrayList<Item>();
	items.add(item1);
	items.add(item2);
	order.setItems(items);

	orderRepository.save(order);	
	Assert.assertEquals(1,orderRepository.count());
	Assert.assertEquals(2,itemRepository.count());

	//代码段1
	itemRepository.save(order);	
	Assert.assertEquals(1,orderRepository.count());
	Assert.assertEquals(2,itemRepository.count());

	//代码段2
	//itemRepository.save(items);
	//Assert.assertEquals(1,orderRepository.count());
	//Assert.assertEquals(2,itemRepository.count());
}

```

在该场景中，我们分别测试如下情况：

1.  没有任何 CascadeType 设置，由测试结果可知，order 可以被保存到数据库，但是两个 Item 却不能。
2.  使用`CascadeType.PERSIST`
    1.  单独在 Order 类的 items 属性上加入`cascade={CascadeType.PERSIST}`, 使用**代码段 1**,order 和 items 都可以被保存到数据库; 使用**代码段 2**，order 和 items 都**不能**被保存到数据库。
    2.  单独在 Item 类的 order 属性上加入`cascade={CascadeType.PERSIST}`，使用**代码段 1**,order 可以被保存到数据库，items 不可以被保存到数据库; 使用**代码段 2**，order 和 items 都可以被保存到数据库。
    3.  在 Order 和 Item 类中都使用`cascade={CascadeType.PERSIST}`，使用**代码段 1**,order 和 items 都可以被保存到数据库; 使用**代码段 2**，order 和 items 都可以被保存到数据库。

由此可以知道，在某个类的属性上使用`cascade={CascadeType.PERSIST}`，对该类进行保存操作时，可以级连保存该类中此属性所对应的对象；而对该类的属性对应的对象进行保存操作时，却不能保存该类（存在外键时，二者都不可保存）。但是如果在该类和该类的属性所对应的类别中同时使用`cascade={CascadeType.PERSIST}`，那么无论是从该类出发进行保存操作，还是从该类的属性对应的对象出发进行保存操作，都可以保存二者。

## 场景 2，删除数据 (`CascadeType.REMOVE`)

现在有这样的场景，客户需要删除一个订单, 那么订单中的订单项也需要一并删除，为了可以实现级连删除的效果，我们使用以下测试代码：

```
private Order order;
private List<Item> items = new ArrayList<Item>();

@Before
public void setUp(){
	order = new Order();
	order.setName("order1");
	Item item1 = new Item();

	item1.setName("item1_order1");
	item1.setOrder(order);

	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);

	items.add(item1);
	items.add(item2);
	order.setItems(items);

	orderRepository.save(order);
	itemRepository.save(items);
}

@Test
public void testDelete(){
	//代码段3
	orderRepository.delete(order);
	Assert.assertEquals(0, orderRepository.count());
	Assert.assertEquals(0, orderRepository.count());

	//代码段4
	itemRepository.delete(items);
	Assert.assertEquals(0, orderRepository.count());
	Assert.assertEquals(0, itemRepository.count());
}

```

在该场景中，我们分别测试如下情况：

1.  在 Order 和 Item 中都没有使用`CascadeType.REMOVE`时，单独删除 order 对象是不成功的，这是由于数据库在 item 表中有基于 order 的外键约束。单独删除 items 可以成功的。
2.  使用`CascadeType.REMOVE`
    1.  在 Order 类的 items 属性上使用`CascadeType.REMOVE`, 使用**代码段 3**，通过 Order 对象的删除操作，可以级连删除 order 中 items 的 Item 对象 (在删除过程中，会先删除 items，然后再删除 order)；但使用**代码段 4**，虽然 items 可以成功删除，但是其关联的 order 对象却不能级连删除。
    2.  在 Item 类的 order 属性上使用`CascadeType.REMOVE`, 使用**代码段 3**, Order 对象的删除操作是失败的，这是因为在存储 item 的表中有引用 order 表的外键；但是使用**代码段 4** 却可以成功的删除 items 及其级连的 order 对象。其过程是先更新 items 中引用的 order 的外键，设置 items 对 order 的引用为空值。然后删除 items，然后再删除 order(注意，如果 items 为多条，那么会先删除一条 item，然后删除 order，然后再删除其余的 item)。
    3.  在 Order 和 Item 中都使用`CascadeType.REMOVE`时，使用**代码段 3** 和**代码段 4** 都可以成功，这表明，在二者都使用`CascadeType.REMOVE`时，既可以通过删除 order，同时级连删除 items；也可以通过删除 items，同时级连删除 order。

通过以上的分析，可以了解到，在一般的业务场景中，需求基本是在删除 order 时同时级连删除 items，但反过来，在删除 items 的时候同时也要求删除 order 却不一定适合业务场景。即使删除了所有和 order 相关的 items，可能也需要保持住那个没有 items 的 order。所以这里的建议是，最好不要在 order 和 item 双方中都同时使用`CascadeType.REMOVE`，即最好不要在关系的维护端 (这里指 Item 类，因为 item 表中会有 order 的外键，所以 item 是关系的维护端) 使用`CascadeType.REMOVE`。

## 场景 3, 更新数据 (`CascadeType.MERGE`)

在业务上，经常会有这样一种类似的需要：查找到了一个业务实体后，要更新该实体，同时也需要更新该实体所关联的其他业务实体。在我们的例子中就是，同时需要更新 Order 和其所关联的 Item。我们使用如下测试代码：

```
private Order order;
private List<Item> items = new ArrayList<Item>();

@Before
public void setUp(){
	order = new Order();
	order.setName("order1");
	Item item1 = new Item();

	item1.setName("item1_order1");
	item1.setOrder(order);

	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);

	items.add(item1);
	items.add(item2);
	order.setItems(items);

	orderRepository.save(order);
	itemRepository.save(items);
}

@Test
public void testUpdate(){
	order.setName("order1_updated");

	items.get(0).setName("item1_order1_updated");
	items.get(1).setName("item2_order1_updated");

	//代码段5
	orderRepository.save(order);
	Assert.assertEquals(1, orderRepository.count(new Specification<Order>(){
		public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "order1_updated");
		}
	}));		
	Assert.assertEquals(1, itemRepository.count(new Specification<Item>() {

		public Predicate toPredicate(Root<Item> root,CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "item1_order1_updated");
		}
	}));

	//代码段6
	itemRepository.save(items);
	Assert.assertEquals(1, itemRepository.count(new Specification<Item>() {
		public Predicate toPredicate(Root<Item> root,CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "item1_order1_updated");
		}
	}));
	Assert.assertEquals(1, orderRepository.count(new Specification<Order>(){
		public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "order1_updated");
		}
	}));		
}

```

在该场景中，我们分别测试如下情况：

1.  在 Order 和 Item 中都没有使用`CascadeType.MERGE`时，使用**代码段 5**, 其测试结果表明更新 Order 成功，但是并没有级连更新 items；使用**代码段 6**, 更新 items 成功，但并没有级连更新 items 所关联的 order 对象。
2.  使用`CascadeType.MERGE`
    1.  单独在 Order 的 items 属性上使用`CascadeType.MERGE`，使用**代码段 5**, 其测试结果表明更新 order 成功，并且级连更新 items 也成功；使用**代码段 6**，其测试结果表明，更新 items 成功，但是并没有级连更新 order。
    2.  单独在 Item 的属性 order 上使用`CascadeType.MERGE`, 使用**代码段 5**，其测试结果表明更新 items 时，可以级连更新其关联的 order 对象；使用**代码段 6**，更新 items 成功，但是并没有级连更新 items 关联的 order 对象。
    3.  在 Order 和 Item 中都使用`CascadeType.MERGE`时，使用**代码段 4** 和**代码段 5** 都可以成功，这表明，在二者都使用`CascadeType.MERGE`时，既可以通过更新 order，同时级连更新 items；也可以通过更新 items，同时更新 order。

通过以上的分析，可以了解到，通过使用`CascadeType.MERGE`, 可以通过更新关系的一端对象，而同时更新关系令一端的数据。

## 场景 4，刷新数据 (`CascadeType.REFRESH`)

这里刷新数据，是对应在这样的业务场景下：对于业务系统，一半会存在多个用户，如果用户 A 取得了 order 和其对应的 items，并且对 order 和 items 进行了修改，同时用户 B 也做了如此操作，但是用户 B 先保存了，然后用户 A 保存时，需要先刷新 order 关联的 items，然后再把用户 A 的变更更新到数据库。这中场景就对应了`CascadeType.REFRESH`的需求。
