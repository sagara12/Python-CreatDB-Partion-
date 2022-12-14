# Python(Partion DB 생성)

## 기능 구현
* 프로그램 시작시 DB가 생성 되어 있는지 판별 
* DB가 생성 되어 있으면 해당 DB를 이용 하고 DB가 없는 상황이면 DB를 생성  
* DB 생성 시점에 Partion(년도)이 된 DB를 생성(3년분)

## 참조 사이트
* https://docs.microsoft.com/ko-kr/sql/relational-databases/partitions/create-partitioned-tables-and-indexes?view=sql-server-ver16 -> DB 파티셔닝

```MsSqL
CREATE PARTITION FUNCTION myRangePF1 (datetime2(0))  
    AS RANGE RIGHT FOR VALUES ('2022-04-01', '2022-05-01', '2022-06-01') ;  
GO  

CREATE PARTITION SCHEME myRangePS1  
    AS PARTITION myRangePF1  
    ALL TO ('PRIMARY') ;  
GO  

CREATE TABLE dbo.PartitionTable (col1 datetime2(0) PRIMARY KEY, col2 char(10))  
    ON myRangePS1 (col1) ;  
GO
```
* https://mozi.tistory.com/351 -> DB 파티셔닝 및 파티션 스키마 생성
```MsSqL
CREATE PARTITION FUNCTION pf_OrderDate ( datetime )
AS RANGE RIGHT
FOR VALUES (
   '1997' /* 1996년도 파티션 */
  ,'1998' /* 1997년도 파티션 */
  ,'1999' /* 1998년도 파티션 */
)
```
```MsSqL
CREATE PARTITION SCHEME ps_OrderDate
AS PARTITION pf_OrderDate
TO ( [primary], [primary], [primary], [secondary] )
```
```MsSqL
CREATE TABLE [dbo].[Orders_Range](
    [OrderID] [int] IDENTITY(1,1) NOT NULL,
    [CustomerID] [nchar](5) NULL,
    [EmployeeID] [int] NULL,
    [OrderDate] [datetime] NULL,
    [RequiredDate] [datetime] NULL,
    [ShippedDate] [datetime] NULL,
    [ShipVia] [int] NULL,
    [Freight] [money] NULL,
    [ShipName] [nvarchar](40) NULL,
    [ShipAddress] [nvarchar](60) NULL,
    [ShipCity] [nvarchar](15) NULL,
    [ShipRegion] [nvarchar](15) NULL,
    [ShipPostalCode] [nvarchar](10) NULL,
    [ShipCountry] [nvarchar](15) NULL
) ON ps_OrderDate ( OrderDate )    /* 파티션 구성표와 파티션 컬럼을 기술 */
GO
```
## 코드 구현
```Pthon
  # 테이블 검색 SQL 쿼리문 작성
        search_talbe_quarry = "select * from sys.tables where name ='"+table_name+"'"
        print(search_talbe_quarry)

        # 데이터 베이스에 해당테이블이 있는지 검색
        search_table=[]
        cur = conn.cursor()
        cur.execute(search_talbe_quarry)
        row = cur.fetchone()

        # 데이테 베이스에 테이블이 없으면 None
        if row is None:

            create_flag = 1


        else:

            create_flag = 2


        #해당하는 테이블이 없을때
        if create_flag == 1:
            # 파티션 함수를 생성
            string_partition_fuc_1 = "CREATE PARTITION FUNCTION pf_LogDate ( datetime )"
            string_partition_fuc_2 = "AS RANGE RIGHT"
            string_partition_fuc_3 = " FOR VALUES ("
            string_partition_fuc_4 = "'2021'"
            string_partition_fuc_5 = ",'2022'"
            string_partition_fuc_6 = ",'2023'"
            string_partition_fuc_7 = ")"

            string_partition_fuc = (string_partition_fuc_1+string_partition_fuc_2+
                                    string_partition_fuc_3+string_partition_fuc_4+string_partition_fuc_5+
                                    string_partition_fuc_6+string_partition_fuc_7)

            print(string_partition_fuc)
            cur.execute(string_partition_fuc)
            conn.commit()


            # 파티션 스키마를 생성
            string_partition_sche1 = "CREATE PARTITION SCHEME ps_LogDate"
            string_partition_sche2 = " AS PARTITION pf_LogDate"
            string_partition_sche3 = " TO([primary], [primary], [primary], [primary])"

            string_partition_sche = (string_partition_sche1 + string_partition_sche2 + string_partition_sche3)

            print(string_partition_sche)
            cur.execute(string_partition_sche)
            conn.commit()

            # 2. 테이블을 생성하는 쿼리문 작성

            string_creat_table1 = "CREATE TABLE [dbo].[Log_History]("
            string_creat_table2 = "[Colum_Number] [int] IDENTITY(1,1) NOT NULL,"
            string_creat_table3 = "[Now_Date] [datetime] NOT NULL,"
            string_creat_table4 = "[Associated_User] [varchar](500) NOT NULL,"
            string_creat_table5 = "[Machine_Name] [varchar](500) NOT NULL,"
            string_creat_table6 = "[Delivery_Group] [varchar](500) NOT NULL,"
            string_creat_table7 = "[Session_Start_Time] [datetime] NOT NULL,"
            string_creat_table8 = "[Session_End_Time] [datetime] NOT NULL,"
            string_creat_table9 = "[Session_Duration(hh:mm)] [varchar](500) NOT NULL,"
            string_creat_table10 = "[Session_Auto_Reconnect_Count] [int] NOT NULL"
            string_creat_table11 = " )ON ps_LogDate ( Now_Date )"

            string_creat_table = (string_creat_table1+string_creat_table2+string_creat_table3+string_creat_table4
                                  +string_creat_table5+string_creat_table6+string_creat_table7+string_creat_table8
                                  +string_creat_table9+string_creat_table10+string_creat_table11
                                  )

            cur.execute(string_creat_table)
            conn.commit()
            cur.close()
            conn.close()
 ```
