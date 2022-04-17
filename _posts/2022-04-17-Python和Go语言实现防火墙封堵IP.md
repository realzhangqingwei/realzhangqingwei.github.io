---
layout:     post
title:     Python和Go语言实现防火墙封堵IP
subtitle:   
date:       2022-04-17
author:     张庆伟
header-img: img/gje.jpg
catalog: true
tags:
    - Python
    - Go
    - 华为
    - 网络设备
    - 防火墙
    - Ensp
---
## 一、目的

使用Ensp搭建网络设备环境, 并用python和Go脚本实现批量封堵IP,  把重复繁琐的工作简单化.

## 二、Ensp环境

### 1、 使用ensp的云, 并且防火墙配置了ssh服务

### 2、拓扑图如下

![20220417154718](https://raw.githubusercontent.com/realzhangqingwei/realzhangqingwei.github.io/master/imgs_for_blogs/20220417154718.png)

## 三、python版本实现IP地址批量封堵

```
from netmiko import ConnectHandler, NetmikoTimeoutException, NetmikoAuthenticationException
from multiprocessing import Process
from datetime import datetime
import logging
import argparse
# import pretty
import os
import sys
########################################################################################
COMMANDS_LIST = [
    "system-view", "dis version", "ip address-set 20220223 type object", "display ip address-set verbose  20220223 item"
]


def parse_args():
    """
    参数化构建程序
    """
    parser = argparse.ArgumentParser(description='Re-evaluate results')
    parser.add_argument('output_dir', nargs=1, help='results directory',
                        type=str)
    parser.add_argument('--imdb', dest='imdb_name',
                        help='dataset to re-evaluate',
                        default='voc_2007_test', type=str)
    parser.add_argument('--comp', dest='comp_mode', help='competition mode',
                        action='store_true')

    if len(sys.argv) == 1:
        parser.print_help()
        sys.exit(1)


    args = parser.parse_args()
    return args

logging.basicConfig(level=logging.DEBUG, filename="test.log")
logger = logging.getLogger("netmiko")


def show_version(a_device):
    '''
    Execute dis version command using Netmiko
    '''
    creds = a_device['credentials']
    remote_conn = ConnectHandler(device_type=a_device['device_type'],
                                 ip=a_device['ip_address'],
                                 username=creds['username'],
                                 password=creds['password'],
                                 port=a_device['port'],
                                 allow_auto_change=True,
                                 auth_timeout=50,
                                 secret='')

    remote_conn.enable()
    remote_conn.find_prompt()
    # remote_conn.send_command("dis cu")
    try:
        print("开始批量执行IP地址封堵,解封动作.")
        remote_conn.send_config_from_file("BlockCmd.txt")
    except Exception as e:
        print("批量执行IP地址封堵,解封动作, 产生的告警信息是: %s." % e)


def Cmd_List():
    CmdList = []
    CmdList.append("ip address-set 20220223 type object" + "\n")
    # 下面是封堵IP的案例的组装命令案例
    with open("BlockIp.txt", "r+") as f1:
        with open("BlockCmd.txt", "w+") as f2:
            for ip in f1:
                cmd = "address" + " " + \
                    ip.strip().split("/")[0] + " " + \
                    "mask" + " " + ip.strip().split("/")[1]
                CmdList.append(cmd + "\n")
            f2.writelines(CmdList)

    # 以下是解封IP的案例, 组装命令, 范围就是解封多少条.
    # with open("BlockCmd.txt", "w+") as f2:
    #         for ip in range(0, 1000):
    #             cmd = "undo" + " " + "address" + " " + str(ip)
    #             CmdList.append(cmd + "\n")
    #         f2.writelines(CmdList)


def main():
    '''
    Use processes and Netmiko to connect to each of the devices in the database. Execute
    'dis version' on each device. Record the amount of time required to do this.
    '''
    Cmd_List()
    start_time = datetime.now()
    # 新增设备, 增加字典
    devices = [{'device_type': 'huawei',
               'ip_address': '192.168.0.35',
                'credentials': {'username': 'hcnp',
                                'password':  'hcnp@12345',
                                },
                'port': 6000,
                'secret': '',
                },
               ]

    procs = []
    for a_device in devices:
        # show_version(a_device)
        my_proc = Process(target=show_version, args=(a_device,))
        my_proc.start()
        procs.append(my_proc)

    for a_proc in procs:
        print(a_proc)
        a_proc.join()

    print("\nElapsed time: " + str(datetime.now() - start_time))


if __name__ == "__main__":
    # args = parse_args()
    # print(args) 
    main()

```



## 四、Go版本实现IP地址封堵

```
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
	"sync"

	"golang.org/x/crypto/ssh"
)

//获取账号和密码的对应关系
type HostPassword struct {
	Host     string
	Username string
	Password string
	Port     string
}

var (
	commands = []string{}   //执行命令组
	hp       []HostPassword //保存账号和密码
	wg       sync.WaitGroup //执行goroutine, 确保goroutine执行完输出到屏幕
)

func main() {
	//1. 选择操作交换机
	// 1.1 输入要执行交换机
	// 创建结构体类型切片, 本次只操作一个主机, 更改长度可操作多个网络设备
	hp = make([]HostPassword, 1)
	hp[0] = HostPassword{Host: "192.168.0.35", Username: "hcnp", Password: "hcnp@12345", Port: "6000"}
	fmt.Println("组装的hp信息是: ", hp)

	// 1.2 输入要执行的初始化命令, 顺便把命令也带进去
	commands = append(commands, "dis version\n", "system-view\n", "ip address-set 20220223 type object\n")
	file, err := os.Open("BlockCmd.txt")
	if err != nil {
		log.Fatalf("Error when opening file: %s", err)
	}
	fileScanner := bufio.NewScanner(file)
	for fileScanner.Scan() {
		commands = append(commands, fileScanner.Text()+"\n")
	}
	if err := fileScanner.Err(); err != nil {
		log.Fatalf("Error while reading file: %s", err)
	}
	file.Close()
	fmt.Println(commands)
	//2. 执行交换机操作
	errors := SshSwitch(hp)
	if errors != nil {
		log.Fatalln(errors)
	}
	// 同步等待, 是把结果输出到屏幕
	wg.Wait()
}

//建立ssh连接
func SshSwitch(hostpasswords []HostPassword) error {
	//循环获取hostpasswords的账号和密码
	for i, _ := range hp {
		//添加同步组，下面会执行goroutin
		wg.Add(1)
		config := &ssh.ClientConfig{
			Config: ssh.Config{
				Ciphers: []string{"aes128-ctr", "aes192-ctr", "aes256-ctr", "aes128-gcm@openssh.com", "arcfour256", "arcfour128", "aes128-cbc", "3des-cbc", "aes192-cbc", "aes256-cbc"},
			}, //添加了很多加密方式，为了应对不同的密码规则
			User: hp[i].Username,
			Auth: []ssh.AuthMethod{
				ssh.Password(hp[i].Password),
			},
			HostKeyCallback: ssh.InsecureIgnoreHostKey(), //此处相当于执行nil，但是并不安全
		}
		addr := fmt.Sprintf("%s:%s", hp[i].Host, hp[i].Port) //组合主机名和端口
		client, err := ssh.Dial("tcp", addr, config)
		if err != nil {
			log.Fatalln("建立ssh连接错误:", err)
			return err
		}
		//执行goroutine，但是没有返回错误。
		go HandleSession(client, commands, &wg)
	}
	return nil
}

//建立session，执行命令。
func HandleSession(client *ssh.Client, commands []string, wg *sync.WaitGroup) error {
	//建立session
	session, err := client.NewSession()
	if err != nil {
		log.Fatalln("创建session出错", err)
		return err
	}
	//延迟关闭session
	defer session.Close()

	//设置terminalmodes的方式
	modes := ssh.TerminalModes{
		ssh.ECHO:          0,     // disable echoing
		ssh.TTY_OP_ISPEED: 14400, // input speed = 14.4kbaud
		ssh.TTY_OP_OSPEED: 14400, // output speed = 14.4kbaud
	}
	//建立伪终端
	err = session.RequestPty("xterm", 80, 40, modes)
	if err != nil {
		log.Fatal("创建requestpty出错", err)
		return err
	}
	//设置session的标准输入是stdin
	stdin, err := session.StdinPipe()
	if err != nil {
		log.Fatal("输入错误", err)
		return err
	}
	//设置session的标准输出和错误输出分别是os.stdout,os,stderr.就是输出到后台
	session.Stdout = os.Stdout
	session.Stderr = os.Stderr
	err = session.Shell()
	if err != nil {
		log.Fatal("创建shell出错", err)
		return err
	}
	//将命令依次执行
	for _, cmd := range commands {
		fmt.Println(cmd)
		_, err = fmt.Fprintf(stdin, "%s\n", cmd)
		if err != nil {
			log.Fatal("写入stdin出错", err)
			return err
		}
	}

	//执行等待
	err = session.Wait()
	if err != nil {
		log.Fatal("等待session出错", err)
		return err
	}
	// 退出ssh
	err = session.Close()
	if err != nil {
		log.Fatal("关闭出错", err)
		return err
	}
	//减少同步组的次数
	wg.Done()
	return nil
}

```


## 五、总结

使用中发现, Go语言的速度比Python快很多, 毕竟python是胶水语言, Go才是未来.

## 六、  参考链接
