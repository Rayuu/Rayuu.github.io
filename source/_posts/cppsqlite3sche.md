---
title: C++使用sqlite3进行简单的课表管理
date: 2016-11-30 21:12:46
tags: c/c++
---

 C++使用sqlite3进行简单的课表管理

## 一、生成sqlite3.lib

1.在sqlite官网 [http://www.sqlite.org/download.html](http://www.sqlite.org/download.html) 上下载`sqlite-amalgamation-3071000.zip` 和`sqlite-dll-win32-x86-3071000.zip`。

2.分别解压上述两个文件到各自文件夹下（`sqlite3.def、sqlite3.dll`在同一文件夹`sqlite-dll`下）。


<!---more--->

3.从VS2010的安装目录下`Microsoft Visual Studio 10.0\VC\bin`找到`lib.exe`和`link.exe`，从VS2010的安装目录下`\Microsoft Visual Studio 10.0\Common7\IDE`找到`mspdb100.dll`。将`lib.exe`` link.exe ``mspdb100.dll`放到步骤2中的`sqlite-dll`文件夹下。

 

4.打开cmd窗口，转到上述sqlite-dll文件夹下，在Windows右键添加cmd菜单可以快速到达，详见[http://blog.rayuu.com/cmdhere.html](http://blog.rayuu.com/cmdhere.html)

5.输入命令：

	LIB /DEF:sqlite3.def /machine:ix86

这时，在sqlite-dll文件夹下会出现sqlite3.lib。

## 二、VS2010静态编译生成exe文件

VS2010静态编译生成的.exe可执行文件，可以免安装在其他电脑直接运行

静态编译：就是在编译可执行文件的时候，将可执行文件需要调用的对应动态链接库（.so）中的部分提取出来，链接到可执行文件中去，使可执行文件在运行的时候不依赖动态链接库。

编译方式：

> 第1种：

设置：

1、项目->配置属性->常规->MFC的使用：在静态库中使用MFC

2、项目 -> 配置属性->C/C++->代码生成->运行库 :选择 多线程调试（/MTd）。

编译时，选择的是debug，win32

然后执行编译生成方案，在该工程目录下的debug文件中，找到该.exe文件，即可在其他电脑运行。

 

> 第2种：

一般可以配置一下两项：

1.项目 -> 配置属性->常规->MFC的使用 :在静态库中使用MFC。

2.项目 -> 配置属性->C/C++->代码生成->运行库 :选择 多线程调试（/MT）。

编译时，选择的是release，win32（这个选择项在工具栏的debug选框中，一般我们使用debug方式）


debug方式产生的文件会比较大，听说它包含了一些调试用的信息，release方式生成的只是该软件所需要的所有功能而已（这个我也不懂，反正大小差不多2:1）。

## 三、课表程序编写



	#include<iostream>
	#include<sstream>
	#include"lib/sqlite3.h"
	#include<string>
	 
	using namespace std;
	//定义课表结构
	struct content
	{
		string context;
	};
	/**********************************
			数据库操作
	***********************************/
	bool Connect();//连接数据库并创建表
	bool Select(int day);//查询单条记录
	bool Delete(int day);//删除单条记录
	bool SelectAll();//查询全部
	bool DeleteAll();//删除全部
	bool Add(int day,const string& no1,const string& no2,const string& no3,const string& no4,const string& no5);//添加单条记录
	int AddDay(content *p);//添加某一天记录
	bool exist_in_db(int pathstr);//判断要添加的数据是否存在
	/**********************************
			其他操作
	***********************************/
	string IntToString (int a);//数字转字符
	void Menu();//菜单
	void show(content *m);//显示输入的课表内容
	void Find();//查找
	void DelDay();//删除某一天的数据
	//SQLITE_OK 是零
	sqlite3 *db=NULL;
	 
	// 回调sql
	static int callback(void *NotUsed, int argc, char **argv, char **azColName){
	   int i;
	   for(i=0; i<argc; i++){
	      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
	   }
	   printf("\n");
	   return 0;
	}
	 
	static int feedback(void *NotUsed, int argc, char **argv, char **azColName){
	   int i;
	   cout<<"星期 "<<argv[1]<<" 的课表为：\n";
	   for(i=2; i<argc; i++){
		  cout<<"第 "<<i-1<<" 节课： ";
	      printf("%s\n", argv[i] ? argv[i] : "NULL");
	   }
	   printf("\n");
	   return 0;
	}
	 
	int main()
	{
		int SetDay;//星期
		int MenuChoice;//菜单选择
		char *errmsg = 0;
		content *p = new content[5];
		Connect();
		Menu();
		while(cin>>MenuChoice)
		{
			cin.sync();
			cin.clear();
			switch(MenuChoice)
			{
			case 1:{
					SetDay = AddDay(p);
					show(p);
					Add(SetDay,p[0].context,p[1].context,p[2].context,p[3].context,p[4].context);
				   }break;
			case 2:{Find();}break;
			case 3:{DelDay();}break;
			case 0:{exit(0);}break;
			default :Menu();break;
			}
			cout << "按回车键返回主菜单......";
			cin.get();
			system("cls");
			Menu();
		}
	    sqlite3_close(db);
		system("pause");
		return 0;
	}
	 
	//connect & create table
	bool Connect()
	{
		int con;
		con = sqlite3_open("schedule.db",&db);
		if(con)
		{
			fprintf(stderr, "Can't open database: %s\n",sqlite3_errmsg(db));
			exit(0);
		}
		else
		{
			fprintf(stdout,"连接数据库成功\n");
		}
		char *sql;
		char * errmsg=0;
		// 新建表 schedule
		sql = "CREATE TABLE IF NOT EXISTS schedule("\
			"ID INTEGER PRIMARY KEY AUTOINCREMENT,"\
			"WEEKLY			INT 	NOT NULL,"\
			"NO1			TEXT	NOT NULL,"\
			"NO2			TEXT	NOT NULL,"\
			"NO3			TEXT	NOT NULL,"\
			"NO4			TEXT	NOT NULL,"\
			"NO5			TEXT	NOT NULL"\
			");";
		con = sqlite3_exec(db,sql,callback, 0, &errmsg);
		if(con != SQLITE_OK)
		{
			cout << "add fail: "<<errmsg<<endl;
			return false;
		}
		return true;
	}
	 
	//添加
	bool Add(int day,const string& no1,const string& no2,const string& no3,const string& no4,const string& no5)
	{
		char *errmsg=0;
		int res=1;
		int  week;
		week = day;
		string strsql = "";
		strsql += "insert into schedule(WEEKLY,NO1,NO2,NO3,NO4,NO5)";
		strsql += " values(";
		strsql += "'";
		strsql += IntToString(day);
		strsql += "','";
		strsql += no1;
		strsql += "','";
		strsql += no2;
		strsql += "','";
		strsql += no3;
		strsql += "','";
		strsql += no4;
		strsql += "','";
		strsql += no5;
		strsql += "');";
		//启动一个事务处理
		//cout <<strsql<<endl;
		if(!exist_in_db(week))
		{
			res = sqlite3_exec(db,strsql.c_str(),0,0, &errmsg); 		
		}
		else
		{
			return false;
		}
		if(res != SQLITE_OK)
		{
			cout << "add fail: "<<errmsg<<endl;
			return false;
		}
		else
		{
			cout << "add success"<<endl;
		}
		return true;
	}
	 
	//查找某一天的课表
	bool Select(int day)
	{
		stringstream strsql;
		char *errmsg = 0;
		//启动一个事务处理
		int res; 
	 
		strsql << "SELECT * FROM schedule WHERE WEEKLY=";
		strsql << day;
		string str = strsql.str();
		res=sqlite3_exec(db,str.c_str(),feedback, 0, &errmsg);
		if(res!=SQLITE_OK)
		{
			cout << "查找失败"<<endl;
			return false;
		}
		return true;
	}
	 
	// 查找全部
	bool SelectAll()
	{
		stringstream strsql;
		char *errmsg = 0;
		//启动一个事务处理
		int res;  
		strsql << "SELECT * FROM schedule";
		string str = strsql.str();
		res=sqlite3_exec(db,str.c_str(),feedback, 0, &errmsg);
		if(res!=SQLITE_OK)
		{
			cout << "查找失败"<<endl;
			return false;
		}
		return true;
	}
	 
	// 删除
	bool Delete(int day)
	{
		stringstream strsql;
		char *errmsg = 0;
		//启动一个事务处理
		int res; 
		strsql << "DELETE FROM schedule WHERE WEEKLY=";
		strsql << day;
		string str = strsql.str();
		res=sqlite3_exec(db,str.c_str(),feedback, 0, &errmsg);
		if(res!=SQLITE_OK)
		{
			cout << "删除失败"<<endl;
			return false;
		}
		return true;
	}
	 
	// 删除全部
	bool DeleteAll()
	{
		stringstream strsql;
		char *errmsg = 0;
		//启动一个事务处理
		int res;  
		strsql << "DELETE FROM schedule";
		string str = strsql.str();
		res=sqlite3_exec(db,str.c_str(),feedback, 0, &errmsg);
		if(res!=SQLITE_OK)
		{
			cout << "删除全部失败"<<endl;
			return false;
		}
		return true;
	}
	 
	//判断星期是否存在
	bool exist_in_db(int pathstr)
	{
	    char sql_query[128]={0};
	    sprintf(sql_query,"select count(*) from schedule where WEEKLY='%d'",pathstr);
	//	sprintf(sql_query,"select count(*) from schedule where ID='%d'",day);
		//cout << sql_query<<endl;
	    sqlite3_stmt *pstmt;
	    sqlite3_prepare(db, sql_query, strlen(sql_query), &pstmt, NULL);
	    sqlite3_step(pstmt);
	    int count=sqlite3_column_int(pstmt,0);
	    sqlite3_finalize(pstmt);
	    //cout << count<<endl;
	    if(count > 0)
		{
			cout<<"星期 "<<pathstr<<" 的课表已添加，\n请删除后重新添加."<<endl;
			return true;
		}
	    return false;
	}
	 
	// 输入课表
	int AddDay(content *p)
	{
		int day;
		cout <<"请输入星期数： "<<endl;
		cin >> day;
		cin.get();
		int i=0;
		for(i=0;i<5;i++)
		{
			cin.clear();
			cin.sync();
			cout << "第 "<<i+1<<" 节课"<<endl;
			getline(cin,p[i].context);
		}
		return day;
	 
	}
	 
	// 删除
	void DelDay()
	{
		int day;
		cout <<"Please enter the day(1~7)"<<endl;
		cin>>day;
		//while(cin>>day)
		{
			cin.sync();
			cin.clear();
			switch(day)
			{
			case 1:Delete(day);break;
			case 2:Delete(day);break;
			case 3:Delete(day);break;
			case 4:Delete(day);break;
			case 5:Delete(day);break;
			case 6:Delete(day);break;
			case 7:Delete(day);break;
			default:DeleteAll();break;
			}
		}
		if(!cin)
		{
			cout << "请输入数字......"<<endl;
		}
	 
	}
	 
	// 显示输入的内容
	void show(content *m)
	{
		cout <<"你输入的内容是："<<endl;
		int i=0;
		for(i=0;i<5;i++)
		{
			cout << "第 "<<i+1<<" 节课 "<< m[i].context << endl;
		}
		cout << endl;
	}
	 
	// 菜单
	void Menu()
	{
		cout<<"+++++++++++++++++++++++++++++++++++++++++\n";
	    cout<<"+               课    表                +\n";
	    cout<<"+                V1.0                   +\n";
	    cout<<"+               by:Rayu                 +\n";
	    cout<<"+             2016.10.18                +\n";
		cout<<"+++++++++++++++++++++++++++++++++++++++++\n";
	    cout<<"\n";
		cout<<"【1】添加课表\t";
		cout<<"【2】查询课表\n";
		cout<<"【3】删除课表\t";
		cout<<"【0】退出系统\n";
		cout<<"输入对应的数字进入菜单： \n";
	}
	 
	void Find()
	{
		int day;
		cout <<"Please enter the day(1~7)"<<endl;
		cin>>day;
		//while(cin>>day)
		{
			cin.sync();
			cin.clear();
			switch(day)
			{
			case 1:Select(day);break;
			case 2:Select(day);break;
			case 3:Select(day);break;
			case 4:Select(day);break;
			case 5:Select(day);break;
			case 6:Select(day);break;
			case 7:Select(day);break;
			default:SelectAll();break;
			}
		}
		if(!cin)
		{
			cout << "请输入数字......"<<endl;
		}
	}
	 
	// int to string
	string IntToString (int a)
	{
	    ostringstream temp;
	    temp<<a;
	    return temp.str();
	}

 
