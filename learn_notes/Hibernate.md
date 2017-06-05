# 调用带参数存储过程，返回实体，此方法为无分页式，适合返回数据量小但是逻辑复杂的查询
##### BaseDAO.java
```java
	public List<T> getObjByUsp(String sql, Class classname, Map<Integer, String> param) {
		try {
			org.hibernate.SQLQuery sqlpro = this.getSession().createSQLQuery(sql);
			for (java.util.Map.Entry<Integer, String> entry : param.entrySet()) {
				sqlpro.setString(entry.getKey(), entry.getValue());
			}
			java.util.List olist = sqlpro.addEntity(classname).list();
			return olist;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
```
##### QueryOption.java
```java
    public StringBuffer initHql() {
        //?的个数要与下面的map数量一致
		StringBuffer hql = new StringBuffer("{Call usp_Name(?,?,?,?)}");
		return hql;
	}

	public Map<Integer, String> initHql_param() {
		//参数统一用String
		Map<Integer, String> map = new LinkedHashMap<>();

		map.put(0, para1);
		map.put(1, para2);
		map.put(2, para3);
		map.put(3, para4 != null ? new SimpleDateFormat("yyyy-MM-dd").format(para4) : "1990-01-01");
		
		return map;
	}
```
##### Service.java
```java
    @Override
	public String queryResult(QueryOption q) throws ServiceException {

		StringBuffer hql = q.initHql();
		Map<Integer, String> param = q.initHql_param();
		List<Obj> list = BaseDAO.getObjByUsp(hql.toString(), Obj.class, param);

		return JSON.toJSONString(CollectionUtil.arrayAsMap("list", list));

	}
```