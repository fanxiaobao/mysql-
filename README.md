# mysql-
记录一些常见mysql相关注意事项

Mysql数据按天分区,定期删除。 
目前需求为：
          1.日志表需要按天分区
　　      2.只保留一个月数据
想到的方案有
          1.创建两个事件,一个事件生成未来需要的分区,另一个事件定期检查过期数据(移除分区)
　　      2.创建事件每小时执行一次,删除事件每天执行一次
　　      3.事件开始时需要先创建一个当前所需分区
        
先构造存储过程 create_partition_today :将表转化为分区表,并将历史数据归集到该分区,未来数据则按天放置:        
#alter table to partition table
DELIMITER $$
USE `dc_log`$$
DROP PROCEDURE IF EXISTS `create_partition_today`$$
CREATE PROCEDURE `create_partition_today`(IN_SCHEMANAME VARCHAR(64), IN_TABLENAME VARCHAR(64))
  BEGIN
    DECLARE BEGINTIME TIMESTAMP;
    DECLARE ENDTIME TIMESTAMP;
    DECLARE DAYS_ENDTIME INT;
    DECLARE PARTITIONNAME VARCHAR(16);
    SET BEGINTIME = NOW();
    SET ENDTIME = BEGINTIME + INTERVAL 1 DAY;
    SET PARTITIONNAME = DATE_FORMAT(BEGINTIME, 'p%Y%m%d');
    SET DAYS_ENDTIME = TO_DAYS(ENDTIME);
    SET @SQL = CONCAT('ALTER TABLE `', IN_SCHEMANAME, '`.`', IN_TABLENAME, '`',
                        ' PARTITION BY RANGE (to_days(create_time))
    (PARTITION ', PARTITIONNAME, ' VALUES LESS THAN (', DAYS_ENDTIME, '))');
    PREPARE STMT FROM @SQL;
    EXECUTE STMT;
    DEALLOCATE PREPARE STMT;
  END$$
DELIMITER ;





