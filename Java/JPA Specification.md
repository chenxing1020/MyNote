1 简介
	Spring Data JPA提供了基础的CRUD功能，但是在进行复杂的条件查询时，就会比较麻烦，官方提供的是继承JpaSpecificationExecutor，然后拼接Specification，实际操作中会造成大量重复的代码，所以本文档对Specification进行封装。
2 JpaSpecificationExecutor
	JPA提供了动态接口，利用类型检查的方式，进行复杂的条件查询，比自己写SQL要安全很多。
1.	public interface JpaSpecificationExecutor<T> {  
2.	    Optional<T> findOne(@Nullable Specification<T> var1);  
3.	    List<T> findAll(@Nullable Specification<T> var1);  
4.	    Page<T> findAll(@Nullable Specification<T> var1, Pageable var2);  
5.	    List<T> findAll(@Nullable Specification<T> var1, Sort var2);  
6.	    long count(@Nullable Specification<T> var1);  
7.	}  
可以看到，Specification是所有方法的入参，实际上它是一个接口，并且只有一个方法：
1.	@Nullable  
2.	Predicate toPredicate(Root<T> var1, CriteriaQuery<?> var2, CriteriaBuilder var3);  
CriteriaBuilder用来构造CriteriaQuery实例；CriteriaQuery被赋予泛型类型，在执行时返回结果的类型，在CriteriaQuery实例上可以设置查询表达式；Root是一个查询表达式，它表示持久化实体的范围，类似于JPQL或SQL查询的From子句；返回结果类型Predicate是计算结果为true或false的常见查询表达式，谓词由QueryBuilder来构造。
CriteriaQuery的查询结构如下图所示。
 
3 封装Specification
3.1 条件表达式接口
	用自定义的接口来模拟系统的条件查询。
1.	public interface Criterion {  
2.	    enum Operator {  
3.	        EQ, IN, LIKE, JOIN_IN, JOIN_EQ  
4.	    }  
5.	    Predicate toPredicate(Root<?> root, CriteriaQuery<?> query, CriteriaBuilder builder);  
6.	}  
3.2 封装Specification
	建立一个自定义的Specification，将多个查询条件用AND连接。
1.	public class Criteria<T> implements Specification<T> {  
2.	    private List<Criterion> criterions = new ArrayList<>();  
3.	    @Override  
4.	    public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder builder) {  
5.	        if (!criterions.isEmpty()) {  
6.	            List<Predicate> predicates = new ArrayList<>();  
7.	            for (Criterion c : criterions) {  
8.	                predicates.add(c.toPredicate(root, query, builder));  
9.	            }  
10.	            if (predicates.size() > 0) {  
11.	                return builder.and(predicates.toArray(new Predicate[predicates.size()]));  
12.	            }  
13.	        }  
14.	        return builder.conjunction();  
15.	    }  
16.	    public void add(Criterion criterion) {  
17.	        if (criterion != null) {  
18.	            criterions.add(criterion);  
19.	        }  
20.	    }  
21.	}  
3.3 条件表达式的实现类
	根据条件的不同，对Criterion接口进行实现。
	在实际需求中，暂且分为简单查询和连表查询两类。
3.3.1 简单查询
1.	@AllArgsConstructor  
2.	public class SimpleCriterionImpl implements Criterion {  
3.	    private String fieldName;  
4.	    private Object value;  
5.	    private Operator operator;  
6.	    @Override  
7.	    public Predicate toPredicate(Root<?> root, CriteriaQuery<?> query, CriteriaBuilder builder) {  
8.	        Expression expression = root.get(fieldName);  
9.	        switch (operator) {  
10.	            case EQ:  
11.	                return builder.equal(expression, value);  
12.	            case IN:  
13.	                CriteriaBuilder.In in = builder.in(expression);  
14.	                if (value instanceof String) {  
15.	                    for (String splitValue : ((String) value).split(",")) {  
16.	                        in.value(splitValue);  
17.	                    }  
18.	                    return in;  
19.	                } else if (value instanceof Collection) {  
20.	                    for (Object itorValue : (Collection) value) {  
21.	                        in.value(itorValue);  
22.	                    }  
23.	                    return in;  
24.	                } else  
25.	                    return null;  
26.	            case LIKE:  
27.	                if (value instanceof String) {  
28.	                    return builder.like(expression, '%' + (String) value + '%');  
29.	                } else  
30.	                    return null;  
31.	            default:  
32.	                return null;  
33.	        }  
34.	    }  
35.	}  
主要是对in的处理，由于业务需求暂时只有两种类型的in查询：用逗号隔开的String串、List集合，所以对入参进行了分类。
3.3.2 连表查询
1.	@AllArgsConstructor  
2.	public class JoinCriterionImpl implements Criterion {  
3.	    private String joinName;  
4.	    private String fieldName;  
5.	    private Object value;  
6.	    private Operator operator;  
7.	    @Override  
8.	    public Predicate toPredicate(Root<?> root, CriteriaQuery<?> query, CriteriaBuilder builder) {  
9.	        switch (operator) {  
10.	            case JOIN_IN:  
11.	                CriteriaBuilder.In in = builder.in(root.join(joinName).get(fieldName));  
12.	                if (value instanceof String) {  
13.	                    for (String splitValue : ((String) value).split(",")) {  
14.	                        in.value(splitValue);  
15.	                    }  
16.	                    return in;  
17.	                } else if (value instanceof Collection) {  
18.	                    for (Object itorValue : (Collection) value) {  
19.	                        in.value(itorValue);  
20.	                    }  
21.	                    return in;  
22.	                } else  
23.	                    return null;  
24.	            case JOIN_EQ:  
25.	                return builder.equal(root.join(joinName).get(fieldName), value);  
26.	            default:  
27.	                return null;  
28.	        }  
29.	    }  
30.	}  
连表查询的逻辑和简单查询差不多，只不过生成root的方式发生了改变。
3.4 生成工厂
根据查询条件的不同有不同的实现类，所以在这里需要写一个工厂方法来生成不同实现类的查询条件。
1.	public class CriterionFactory {  
2.	    public static SimpleCriterionImpl eq(String fieldName, Object value, boolean ignoreNull) {  
3.	        if (ignoreNull && isLegal(value)) {  
4.	            return new SimpleCriterionImpl(fieldName, value, Criterion.Operator.EQ);  
5.	        }  
6.	        return null;  
7.	    }  
8.	    public static SimpleCriterionImpl in(String fieldName, Object value, boolean ignoreNull) {  
9.	        if (ignoreNull && isLegal(value)) {  
10.	            return new SimpleCriterionImpl(fieldName, value, Criterion.Operator.IN);  
11.	        }  
12.	        return null;  
13.	    }  
14.	  
15.	    public static SimpleCriterionImpl like(String fieldName, Object value, boolean ignoreNull) {  
16.	        if (ignoreNull && isLegal(value)) {  
17.	            return new SimpleCriterionImpl(fieldName, value, Criterion.Operator.LIKE);  
18.	        }  
19.	        return null;  
20.	    }  
21.	    public static JoinCriterionImpl joinin(String joinName, String fieldName, Object value, boolean ignoreNull) {  
22.	        if (ignoreNull && isLegal(value)) {  
23.	            return new JoinCriterionImpl(joinName, fieldName, value, Criterion.Operator.JOIN_IN);  
24.	        }  
25.	        return null;  
26.	    }  
27.	    public static JoinCriterionImpl joineq(String joinName, String fieldName, Object value, boolean ignoreNull) {  
28.	        if (ignoreNull && isLegal(value)) {  
29.	            return new JoinCriterionImpl(joinName, fieldName, value, Criterion.Operator.JOIN_EQ);  
30.	        }  
31.	        return null;  
32.	    }  
33.	    private static boolean isLegal(Object value) {  
34.	        if (value instanceof String)  
35.	            return StringUtils.isNotBlank((String) value);  
36.	        else if (value instanceof Collection)  
37.	            return !CollectionUtils.isEmpty((Collection) value);  
38.	        else return (value == null) ? false : true;  
39.	    }  
40.	  
41.	}  
这边在工厂方法里面加入了入参的校验，来检查value值的合法性。
3.5 使用
1.	Criteria<Asset> criteria = new Criteria<>();  
2.	criteria.add(CriterionFactory.eq("tenantId", tenantId, true));  
3.	criteria.add(CriterionFactory.in("orgId", filteredOrgs, true));  
4.	criteria.add(CriterionFactory.like("name", searchContent, true));  
5.	criteria.add(CriterionFactory.joinin("tagSet", "id", tags, true));  
6.	return assetRepository.findAll(criteria, pageable);
4 参考文档
https://blog.csdn.net/tianyaleixiaowu/article/details/72876732
https://blog.wuwii.com/jpa-specification.html
https://blog.csdn.net/mine_song/article/details/71248628
