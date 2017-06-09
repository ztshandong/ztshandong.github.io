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
	public List<T> getObjByXml(String sql, Map<Integer, String> param) {
		try {
			Query pro = this.getSession().getNamedQuery(sql);
			for (Map.Entry<Integer, String> entry : param.entrySet()) {
				// pro.setParameter(entry.getKey(), entry.getValue());
				pro.setString(entry.getKey(), entry.getValue());
			}
			List<T> olist = pro.list();
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
		//参数统一用Integer, String
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
# 联表查询
##### tmsck
```java
tmsck.hbm.xml
<!-- 一个中转单号可对应多个出库单号，相当于Tmszz是主表，Tmsck是从表，Tmsck外键列为docNo,默认关联Tmszz的id ，会返回整个Tmszz -->
<!-- 如果Tmszz外键不是id列，例如用supplieCode，则此处增加 property-ref="supplieCode" ，Tmszz的supplieCode字段增加unique="true" -->
<!-- 只设置many-to-one就不返回DocNo字段，要靠数据库设置外键更新此列，但这一列又不是外键，所以还要有docNo对应 -->
<!-- DocNo对应多个字段，这里update与insert要设置为false -->
<!-- 如果不设置Tmszz的话unique要设置为false-->
<hibernate-mapping>
		<class name="com.firs.api.tms.pojo.Tmsck" table="tb_TMS_CKs">
		<id name="id" type="java.lang.Integer">
			<column name="isid" />
			<generator class="identity" />
		</id>
		<property name="docType" type="string">
			<column name="DocType" length="20" />
		</property>
		<property name="docNo" type="string">
			<column name="DocNo" length="32" />
		</property>
		<many-to-one name="tmszzs"
                class="com.firs.api.tms.pojo.Tmszz"
                unique="false" 
                column="DocNo"
                not-found="ignore"
                fetch="join"
                lazy="false"
                not-null="false"
                update="false"
                insert="false"
                />
         <many-to-one name="tmsdbs"
                class="com.firs.api.tms.pojo.Tmsdb"
                unique="false" 
                column="DocNo"
                not-found="ignore"
                fetch="join"
                lazy="false"
                not-null="false"
                update="false"
                insert="false"
                />
		</class>
		<sql-query name="usp_TMS_Search4Cks" callable="true">  
	<query-param name="zzno" type="string"/>  
     <return alias="cks" class="com.firs.api.tms.pojo.Tmsck">  
     <return-property name="id" column="isid" />  
     <return-property name="docType" column="DocType" />  
     <return-property name="docNo" column="DocNo" />  
     <return-property name="ydno" column="YDNO" />  
     <return-property name="tmszzs" column="DocNo" />  
	 <return-property name="tmsdbs" column="DocNo" />  
     </return>  
     {call usp_TMS_Search4Cks(?)}  
   </sql-query> 
</hibernate-mapping>
sql-query还有另一种写法
 <sql-query name="usp_TMS_Search4Cks" callable="true">  
     <query-param name="zzno" type="string"/>  
     <return alias="cks" class="com.firs.api.tms.pojo.Tmsck">  
     <return-property name="id" column="isid" />  
     <return-property name="docType" column="DocType" />  
     <return-property name="docNo" column="DocNo" />  
     <return-property name="ydno" column="YDNO" />  
     <return-property name="tmszzs" column="DocNo" />  
     </return>  
     <return-join alias="zz" property="cks.tmszzs"/>
      SELECT cks.isid,cks.DocType,cks.DocNo,cks.YDNO FROM dbo.tb_TMS_CKs cks LEFT JOIN tb_TMS_ZZ zz ON cks.DocNo=zz.ZZNO
      WHERE cks.DocNo= :zzno
   </sql-query> 
   getNamedQuery的传参格式为pro.setString("zzno", "SHZZ001");传参的Map要修改
tmsck.java
@ManyToOne
// 一个中转单号可对应多个出库单号，相当于中转单是主表，出库单是从表，也就是说关联查询出来的Tmszz是唯一的，所以这里不能用List
private Tmszz tmszzs;
```
## 建议做单向关联，就是Tmsck设置many-to-one之后不要再设置Tmszz的one-to-many
##### tmszz
```java
tmszz.hbm.xml
<class name="com.firs.api.tms.pojo.Tmszz" table="tb_TMS_ZZ">
		<id name="zzno" type="java.lang.String">
			<column name="ZZNO" length="30" />
			<generator class="assigned" />
		</id>
		<property name="carNo" type="string">
			<column name="CarNo" length="20" />
		</property>
<!-- OneToMany，Tmszz外键貌似只能是id，未找到修改的办法，用的少是不是因为这个？ -->
		<set name="tmscks" table="tb_TMS_CKs" 
				inverse="true" lazy="false" fetch="join">
            <key>
                <column name="docNo" not-null="false" />
            </key>
            <one-to-many class="com.firs.api.tms.pojo.Tmsck" />
        </set>
	</class>
</hibernate-mapping>
tmszz.java
@OneToMany
private Set<Tmsck> tmscks;
```
##### service.java
```java
	@Override
	public String tmszz_query4bill(TmsQueryOption q) throws ServiceException {
		// StringBuffer hql = q.initHql_zz4bill();
		// 原生SQL语句
		StringBuffer hqlsql = new StringBuffer(
				" from tb_TMS_ZZ as zz left join (select * from tb_TMS_CKs where docType='ZZ' ) as ck  
				on zz.zzno=ck.docNo where 1=1 ");
		int totalSizesql = tmszzDAO.count4SQL(hqlsql.toString());
		hqlsql.insert(0, " select zz.* ");
		hqlsql.append(" order by zz.DocDate desc ");
		@SuppressWarnings("rawtypes")
		Page pagesql = new Page(q.getPageNo(), totalSizesql, Constants.PAGESIZE_MED, hqlsql.toString());
		StringBuffer hql = new StringBuffer();
		// 格式化伪智能，注释掉之后会自动换行
		// @ManyToOne 设置ck，with后面的条件不可以用ck的，只能用zz的
		StringBuffer hql2 = new StringBuffer(" from Tmsck as ck right join ck.tmszzs as zz  where 1=1 "
				+ " and zz.zzno in (select z.docNo from Tmsck z) and zz.zzno='ZZ001'");
		// @OneToMany
		// 设置zz，不需要wrapTmszz4bill就有tmsck表信息，HQL语法join之后不能跟on，要直接跟where
		// 生成的SQL语句会根据xml的配置自动加上on，on如果有其他条件通过with添加
		StringBuffer hql3 = new StringBuffer(
				" from Tmszz as zz left join zz.tmscks as ck with ck.docType='ZZ'  where 1=1 "
						+ " and zz.zzno in (select z.docNo from Tmsck z) and zz.zzno='ZZ001'");
		// 子查询，不需要设置表之间的关系，只能用在where与select中
		StringBuffer hql4 = new StringBuffer(
				" from Tmszz as zz where 1=1 and zz.zzno in (select z.docNo from Tmsck z where z.docType='ZZ')"
						+ " and zz.zzno='ZZ001'");
		StringBuffer hql5 = new StringBuffer(" from Tmszz as zz where 1=1 and zz.zzno='ZZ001'");
		StringBuffer hql6 = new StringBuffer(
				" from Tmszz as zz where 1=1 and zz.zzno not in (select z.docNo from Tmssettledoc z )");
		hql = hql6;
		int totalSize = tmszzDAO.count(hql.toString());
		hql.insert(0, " select distinct zz ");// 如果不设置表映射只用子查询这个就可以省略
		hql.append(" order by zz.docDate desc ");
		@SuppressWarnings("rawtypes")
		Page page = new Page(q.getPageNo(), totalSize, Constants.PAGESIZE_MED, hql.toString());
		page = tmszzDAO.getPage(page);
		return JSON.toJSONString(CollectionUtil.arrayAsMap("page", wrapTmszz4bill(page)));
		// 原生sql
		// pagesql = tmszzDAO.getPage4SQL(pagesql, Tmszz.class);
		// return JSON.toJSONString(CollectionUtil.arrayAsMap("page",
		// wrapTmszz4bill(pagesql)));
	}
	@SuppressWarnings({ "rawtypes", "unchecked" })
	Page wrapTmszz4bill(Page page) {
		List<Tmszz> zzs = (List<Tmszz>) page.getList();
		for (Tmszz zz : zzs) {
			// 使用xml中配置的存储过程
			StringBuffer hql = new StringBuffer("usp_TMS_Search4Cks");
			Map<Integer, String> param = new LinkedHashMap<>();
			//如果查询用的是xml的第二种格式这里要注意修改Map
			param.put(0, zz.getZzno());
			List list = tmsckDAO.getObjByXml(hql.toString(), param);//
			// List<Tmsck> list1 = tmsckDAO.getObjByXml(hql.toString(), param);
			//此例中的hibernate的session取不到存储过程的out参数值，所以如果需要返回值的话就这样迂回一下
			//循环中如果有这样的需求应该是半夜统一计算，延迟一天出结果
			 List list = tmszzDAO.getObjByUsp(hql.toString(), param);
			 double payable = FormatUtil.parseDoubleValue(list.get(0), 0d);
			 ArrayList<Double> a = (ArrayList<Double>) list;
			 zz.setAmountBalance(payable);
		}
		page.setList(zzs);
		return page;
	}
```