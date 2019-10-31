1、if条件如果需要用到对象的属性 ： 

```sql
	<select id="getComAndMedia" resultType="com.infinite.ssp.modules.ad.vo.AdGraphicalVO" parameterType="com.infinite.ssp.modules.ad.vo.AdGraphicalVO">
		SELECT
		m.id mediaId,
		m.title mediaName,
		c.com_id comId,
		c.com_name comName
		FROM base_company_info c,media_info m
		WHERE
		m.company_id = c.com_id <if test="comId != 0"> AND m.company_id = #{comId}</if>
</select>
	
	-- 上面的test内容，直接使用 AdGraphicalVO 的属性名即可。
```

