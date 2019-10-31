## 1、if、where、set连接语句

```sql
<select id="ifTest" resultType="com.sl.po.Product">
        select * from products where 
        <if test="ProductName!=null">
            name like #{ProductName}
        </if>
        <if test="description!=null">
            and description like CONCAT(CONCAT('%', #{Description, jdbcType=VARCHAR}),'%')
        </if>
</select>

-- 上面的if，如果第一个失败，第二个成功，会变成 where and description ，会报错，所以要用<where>，自动删除多余开头的 and 和 or.

<select id="whereTest" resultType="com.sl.po.Product">
        select * from products
        -- where标签自动移除第一个and 
        <where>
            <if test="Name!=null">
                and name like #{Name}
                <!--name like #{Name}-->
            </if>
            <if test="description!=null">
                and description like #{Description}
            </if>
        </where>
</select>

-- 同理，update语句的 <set>避免更新全部为空
<!-- if + set 实现按条件更新-->
    <update id="setTest">
        update products
       -- set标签将移除最后一个“,” 
        <set>
            <if test="cityCode!=null">
              citycode = #{cityCode} ,
            </if>
            <if test="Name!=null">
               name = #{Name} ,
            </if>
            <if test="description!=null">
                description = #{Description} ,
            </if>
        </set>
        where id =#{id}
</update>




```



## 2、trim

​	自定义的，在前缀or后缀加上自己想要的内容。对应prefix 、suffix。

​	也可覆盖掉头部或尾部的内容，对应 prefixOverrides 和 suffixOverrides。

```sql
<select id="trimwhereTest" resultType="com.sl.po.Product">
        select * from products
        -- 自动在前面添加where,并且自动移除位于首部的 AND 或者 OR
        -- !!!!注意此例中的空格也是必要的 AND |OR 
        <trim prefix="WHERE" prefixOverrides="AND |OR">
            <if test="Name!=null">
                and name like #{Name}
            </if>
            <if test="description!=null">
                and description like #{Description}
            </if>
        </trim>
    </select>
```



## 3、choose(选择)

​	类似 if - else  , when - otherwise

```sql
<!-- choose + when + otherwise 只能选择一个作为查询条件 作用类似sql case when then -->
    <select id="choosewhenotherwiseTest" resultType="com.sl.po.Product">
        select * from products
     <where>
        <choose>
            <when test="name!=null">
                and name like #{Name}
            </when>
            <when test="description!=null">
                and description like #{Description}
            </when>
            <otherwise>
                and unitprice > #{UnitPrice}
            </otherwise>
        </choose>
     </where>
   </select>
```



## 4、foreach 循环

​	对一个集合的循环，如果是map，则index是键，item是值

```sql
-- 只有一个List参数时它的参数名为list，即collection="list" ;  如果参数类型时数组object[],则collection="array"

-- index是迭代过程中，迭代的位置
-- collection要遍历的集合的名字
-- separator是连接每个项目的标识符
-- open是首部添加的东西，close是尾部添加的

<select id="foreachTest" resultType="com.sl.po.Product">
      select * from products 
      <where>
        <if test="list!=null">
          <foreach item="id" index="index"  collection="list" open="id in(" separator="," close=")">#{id}</foreach>
        </if>
      </where>
</select>

```



## 5、sql片段

​	重复利用，需要之间include

```sql
<select id = "sqlTest" resultType="com.lingz.po.Product">
		select * from products 
		<where>
		-- 引用sql片段 
        	<include refid="sqltemp"/>
		</where>
</select>

-- 定义sql片段 ：将where条件提取 
    <sql id="sqltemp">
        <if test="cityCode!=null">
                   and citycode = #{cityCode}
                </if>
                <if test="Name!=null">
                    and name like #{Name}
                </if>
                <if test="description!=null">
                    and description like #{Description}
                </if>
    </sql>


```





















