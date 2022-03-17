## 测试数据库的TPS值

```python
import time
import sys
import mysql.connector

from threading import Thread
loops = {}

# 进行sql  tps测试 每秒执行时间
class Tps(Thread) :
    global loops
    # 线程名, 结束时间
    def __init__(self, tablename,endtime,username,password,dbhost,dbname):  # 可以通过初始化来传递参数
        super(Tps, self).__init__()
        self.tablename = tablename
        self.endtime = endtime
        self.username = username
        self.password = password
        self.dbhost = dbhost
        self.dbname = dbname

    def run(self):
        type = 1
        iet = 1
        db = mysql.connector.connect(user=self.username, password=self.password , host= self.dbhost, database=self.dbname,buffered = True)
        self.cnx = db.cursor()
        global loops
        while type == 1 :
           iet +=1
           sql = "insert into " + self.tablename + " values ("+str(iet)+","+str(iet)+",'aa','bb','"+time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+"')"
           self.cnx.execute(sql)
           db.commit()
           self.cnx.execute("update " + self.tablename + " set c3='k"+str(iet)+"' where c1="+str(iet))
           db.commit()
           self.cnx.execute("select * from " + self.tablename + " where c1 ="+str(iet))
           db.commit()
           self.cnx.execute("delete from " + self.tablename + " where c1="+str(iet))
           db.commit()
           key = int(time.time())
           #验证结束时间
           if key > endtime :
             type = 0
           key = str(key)
           if key in loops :
             loops[key] = (loops[key]+4)
           else:
              temp = {key: 4}
              loops.update(temp)
        db.close()
        print("===="+self.tablename+" test end")



if __name__ == "__main__":
     threadnum = 10 # 线程数
     executiontime = 10 # 执行时间秒
     dbhost = '127.0.0.1'    #数据库主机地址
     port = 3306             #数据库端口号
     username = "xxxx"   #数据库登录用户名
     password = "xxxx" #密码
     dbname = "xxxx" #数据库名称
     tablename = "test_tps"
     cnx = mysql.connector.connect(user=username, password=password, host=dbhost, database=dbname)
     cursor = cnx.cursor()
     #执行建表操作
     for  ie in range(threadnum):
         cursor.execute("DROP TABLE  "+ tablename+"_"+str(ie))
         cursor.execute("create table " +tablename+"_"+str(ie) + "(c1 int, c2 int, c3 varchar(100),c4 char(100), c5 DATE) ENGINE=InnoDB ")
     cnx.close()
     #创建索引
     #cursor.execute("create index idx_1_" +tablename + " on "+ tablename+ "  (c1) ")
     #cursor.execute("create index idx_2_" +tablename + " on "+ tablename+ "  (c1,c2.c3) ")
     #结束时间
     endtime = int(time.time())+executiontime
     #创建线程执行测试
     for  ie in range(threadnum):
        tps = Tps( tablename+"_"+str(ie) ,endtime,username,password,dbhost,dbname)
        tps.start()
     while time.time() < endtime :
        time.sleep(0.2)
     for iet in loops:
        print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(int(iet))) +"     "+str(loops[iet]))

```