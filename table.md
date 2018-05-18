## mysql中int时间的转换方法from_unixtime(时间,转换格式) eg:

```mysql
update archives  set 字段A =5 where from_unixtime(pubdate,'%Y-%m-%d %h:%i:%m')='2012-11-11 00:00:00';
```

## 创建mysql储存过程或函数使用

```mysql
create procedure crontab_report_gift_order(in iday int(11))
begin
  declare beg_date int(11) default 0;
	declare end_date int(11) default 0;
  set beg_date = unix_timestamp(iday);
	set end_date = beg_date + 86400;

  replace into report_gift_order(day,gid,golds,money,num,users)
  select iday,gid,sum(golds) as golds,sum(golds) as money,sum(num) as num,count(uid) as uid
  from yly_gift_order where state = 2 and (atime>=beg_date and atime<end_date) group by gid;
end
```

```mysql
  `day` int(11) unsigned NOT NULL COMMENT '天',
  `gid` int(11) NOT NULL DEFAULT '0' COMMENT '商品',
  `golds` int(11) NOT NULL DEFAULT '0' COMMENT '兑换荷豆',
  `money` int(11) NOT NULL DEFAULT '0' COMMENT '兑换钱',
  `num` int(11) NOT NULL DEFAULT '0' COMMENT '兑换次数',
  `users` int(11) NOT NULL DEFAULT '0' COMMENT '兑换用户数',
```

```mysql
CREATE DEFINER=`puroot`@`%` PROCEDURE `crontab_report_invite_page_count`(IN iday int(11))
    SQL SECURITY INVOKER
BEGIN
	declare beg_date int(11) default 0;
	declare end_date int(11) default 0;

	set beg_date = unix_timestamp(iday);
	set end_date = beg_date + 86400;

	replace into report_invite_page_count(day,page,uid,enters,downs,users,installs)
	select iday,page,uid,sum(shows) as shows,sum(downs) as downs,count(distinct ip) as users, 0
    from log_invite_page where day=iday group by page,uid;

    replace into report_invite_page_ad_count(day,adid,uid,clicks,shows,users)
	select iday,adid,uid,sum(clicks) as clicks,sum(shows) as shows,count(distinct ip) as users
    from log_invite_page_ad where day=iday group by adid,uid;

    replace into invite_page_ad_user(adid,uid,clicks,shows,users)
	select adid,uid,sum(clicks) as clicks,sum(shows) as shows,count(distinct ip) as users
    from log_invite_page_ad group by adid,uid;

    update invite_page as a inner join (
		select page,uid, count(distinct ip) as users
		from log_invite_page group by page, uid
    ) as b on a.page=b.page and a.uid=b.uid
    set a.users=b.users;

    update invite_page_ad as a inner join (
		select adid, count(distinct ip) as users
		from log_invite_page_ad group by adid
    ) as b on a.adid=b.adid
    set a.users=b.users;

    replace into report_prize_code_count(day,page,uses)
    select iday,'gws',count(code) from prize_code_gws where `use`=1 and utime>=beg_date and utime<end_date;

    replace into report_prize_order_count(day,pid,prizes,users)
    select iday, pid, count(oid), count(distinct unionid) from prize_order where ltime>=beg_date and ltime<end_date
    group by pid;
END
```

## table














