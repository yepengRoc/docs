表添加字段

alter table add column 表数据量大的话，锁表，不可行



例如需要在线上表A 增加一个字段field

“新表+触发器+迁移数据+rename”方案(pt-online-schema-change)

以user(uid, name, passwd)

扩展到user(uid, name, passwd, age, sex)为例

基本原理是：

(1)先创建一个扩充字段后的新表user_new(uid, name, passwd, age, sex)

(2)在原表user上创建三个触发器，对原表user进行的所有insert/delete/update操作，都会对新表user_new进行相同的操作

(3)分批将原表user中的数据insert到新表user_new，直至数据迁移完成

(4)删掉触发器，把原表移走(默认是drop掉)

(5)把新表user_new重命名(rename)成原表user

扩充字段完成。

优点：整个过程不需要锁表，可以持续对外提供服务

操作过程中需要注意：

(1)变更过程中，最重要的是冲突的处理，一条原则，以触发器的新数据为准，这就要求被迁移的表必须有主键(这个要求基本都满足)

(2)变更过程中，写操作需要建立触发器，所以如果原表已经有很多触发器，方案就不行(互联网大数据高并发的在线业务，一般都禁止使用触发器)

(3)触发器的建立，会影响原表的性能，所以这个操作建议在流量低峰期进行

pt-online-schema-change是DBA必备的利器，比较成熟，在互联网公司使用广泛。

