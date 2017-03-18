title: beego upsert 方法原理
date: 2016-12-13
tags: ["golang","beego"]
categories:
  开源项目
---

## 前言 ##
* 在beego1.6.1版本orm中并未提供insertOrUpdate，但是自己做项目时遇到了这个需求，顾写了一个自己的实现，暂只支持mysql与postgres。实现原理是数据自带可实现insertorupdate的功能语句。
mysql：**`ON DUPLICATE KEY UPDATE`**
postgres : **`ON CONFLICT DO UPDATE SET`**
然后去orm实现中自己拼装sql语句

## code ##
* 好了，亮代码：

```
func (d *dbBase) InsertOrUpdate(q dbQuerier, mi *modelInfo,ind reflect.Value, tz *time.Location, dn string, args ...string) (int64, error) {

	iouStr := ""
	mysql := "mysql"
	postgres := "postgres"
	argsMap := map[string]string{}

	if dn == mysql {
		iouStr = "ON DUPLICATE KEY UPDATE"
	} else if dn == postgres && len(args) > 0 {
		args0 = args[0]
		iouStr = fmt.Sprintf("ON CONFLICT (%s) DO UPDATE SET", args0)
	} else {
		return 0, fmt.Errorf("`%s` nonsupport insert or update in beego", dn)
	}

	for _, v := range args {
		kv := strings.Split(v, "=")
		if len(kv) == 2 {
			argsMap[kv[0]] = kv[1]
		}
	}

	isMulti := false
	names := make([]string, 0, len(mi.fields.dbcols)-1)
	Q := d.ins.TableQuote()
	values, err := d.collectValues(mi, ind, mi.fields.dbcols, true, true, &names, tz)
	if err != nil {
		return 0, err

	marks := make([]string, len(names))
	updateValues := make([]interface{}, 0)
	updates := make([]string, len(names))
	var conflitValue interface{}
	for i, v := range names {
		marks[i] = "?"
		valueStr := argsMap[v]
		if v == args0 {
			conflitValue = values[i]
		}

		if valueStr != "" {
			switch dn {
			case mysql:
				updates[i] = v + "=" + valueStr
				break
			case postgres:
				if conflitValue != nil {
			updates[i] = fmt.Sprintf("%s=(select %s from %s where %s = ? )", v, valueStr, mi.table, args[0])
					updateValues = append(updateValues, conflitValue)
				} else {
					return 0, fmt.Errorf("`%s` must be in front of `%s` in your struct", args[0], v)
				}
				break
			}
		} else {
			updates[i] = v + "=?"
			updateValues = append(updateValues, values[i])
		}

	values = append(values, updateValues...)
	sep := fmt.Sprintf("%s, %s", Q, Q)
	qmarks := strings.Join(marks, ", ")
	qupdates := strings.Join(updates, ", ")
	columns := strings.Join(names, sep)
	multi := len(values) / len(names)

	if isMulti {
		qmarks = strings.Repeat(qmarks+"), (", multi-1) + qmarks
	}
	query := fmt.Sprintf("INSERT INTO %s%s%s (%s%s%s) VALUES (%s) %s "+qupdates, Q, mi.table, Q, Q, columns, Q, qmarks, iouStr)

	if isMulti || !d.ins.HasReturningID(mi, &query) {
		res, err := q.Exec(query, values...)
		if err == nil {
			if isMulti {
				return res.RowsAffected()
			}
			return res.LastInsertId()
		}
		return 0, err
	}

	row := q.QueryRow(query, values...)
	var id int64
	err = row.Scan(&id)
	return id, err
}
```
* 这就是实现功能的全部逻辑，当然要想在beego orm中使用insertorupdate还有一些其他的工作要做，首先这段代码应该添加在 **`beego/orm文件夹下的db.go文件中`**

* 然后在 **`orm.go`** 文件中添加

```
 func (o *orm) InsertOrUpdate(md interface{},colConflitAndArgs ...string) (int64, error) {

	mi, ind := o.getMiInd(md, true)
	id, err := o.alias.DbBaser.InsertOrUpdate(o.db, mi, ind, o.alias.TZ, o.alias.DriverName, colConflitAndArgs...)
	if err != nil {
		return id, err
		}
	o.setPk(mi, ind, id)
	return id, nil
}
```

```
再在types.go文件中的type Ormer interface和type dbBaser interface中分别添加
InsertOrUpdate(md interface{}, colConflitAndArgs ...string) (int64, error) 与
InsertOrUpdate(dbQuerier, *modelInfo, reflect.Value, *time.Location, string, ...string) (int64, error)
```
* 好了，现在大功告成了。可以使用InssertOrUpdate功能了
列如：

### mysql ###
```
mysql：

func IOUFinish(all *Finish) int64 {

	db := orm.NewOrm()
	db.Using("mysql")
	r, e := db.InsertOrUpdate(all, "step=step+1")
	if e != nil {
		fmt.Println(e)
		return 0
	}
	fmt.Println(r)
	return r
}
```
* 这个函数在出入数据时有主键或者唯一键冲突，将执行update操作，其中step列执行+自增操作，其他列按model中的值进行update操作。其中"step=step+1"格式数据可以有多个也可以没有，这种格式只用于自增操作

### postgresql ###
```
postgres：

func IOUFinish(all *Finish) int64 {

	db := orm.NewOrm()
	db.Using("postgres")
	r, e := db.InsertOrUpdate(all,"confilctColumnName" "step=step+1")
	if e != nil {
		fmt.Println(e)
		return 0
	}
	fmt.Println(r)
	return r
}
```
* 当操作postgres数据库是，必须在model后的第一个参数指定你预期的冲突列的列名(由于实现此功能的sql语句需要且数据库版本必须大于9.5，因为实现的语句由9.5版本推出)，其他与mysql一致。

提示：在使用自增操作是最好不要自增主键或者唯一键，可能会引起错误。
