	# /etc/keepalived/keepalived.conf
	# 完整的 keepalived 配置文件由3部分组成：全局定义部分，vrrp实例定义部分，虚拟服务器定义部分
	
	vrrp_instance VI_1 {
	! Configuration File for keepalived
	# 全局定义部分
	global_defs {
	   notification_email {
		test1@163.com	# 设置报警邮件地址，可设置多个
		test2@163.com	# 每行1个; 开启邮件报警，需要开启本机 sendmail 服务
	   }
	   notification_email_from test0@163.com 	# 设置 邮件的发送地址
	   smtp_server smtp.163.com				 	# 设置 smtp server 地址
	   smtp_connect_timeout 30					# 设置 连接 smtp server的超时时间
	   router_id LVS_DEVEL-1					# 设置 keepalived 服务器的一个标识。发邮件时显示在邮件主题中的信息
	}
	# vrrp 实例部分定义
	vrrp_instance VI_1 {
	    state BACKUP							# 指定 keepalived 的角色，可选值：MASTER|BACKUP 分别表示（主|备）
	    interface eth0							# 指定 HA 检测网络的接口
	    virtual_router_id 51					# 虚拟路由表示，是一个数字，同一个vrrp 实例使用唯一的标识
												# MASTER和BACKUP 的 同一个 vrrp_instance 下 这个标识必须保持一致
	    priority 100							# 定义优先级，数字越大，优先级越高。
												# 同一个 vrrp_instance 下，MASTER 优先级必须大于 BACKUP
	    advert_int 1							# 设定 MASTER 与 BACKUP 负载均衡之间同步检查的时间间隔，单位为秒
	    authentication {						# 设置验证类型和密码
	        auth_type PASS						# 设定验证类型，主要有: PASS 和 AH 两种
	        auth_pass 1111						# 设置验证密码，在一个 vrrp_instance 下, MASTER 和 BACKUP必须使用相同的密码才能通信
	    }
	    virtual_ipaddress {						# 设置虚拟IP地址，可以设置多个虚拟IP地址，每行一个
	        192.168.200.16
	        192.168.200.17
	        192.168.200.18
	    }
	}
	# 虚拟服务器定义部分
	virtual_server 192.168.200.100 443 {	# 设置虚拟服务器需要指定虚拟IP地址和服务端口，IP和端口之间用空格隔开
	    delay_loop 6						# 设置运行情况检查，
	    lb_algo rr							# 设置负载均衡调度算法，这里设置为rr, 即轮询算法
	    lb_kind NAT							# 设置LVS实现负载均衡的机制，有 NAT，TUN和DR三个模式可选
	    nat_mask 255.255.255.0
	    persistence_timeout 50				# 会话保持时间，单位是秒。这个选项对动态网页比较有用，为集群系统中的session共享提供了很好的解决方案
											# 用户请求会一直发布到某个服务节点，知道超过这个会话的保持时间。
											# 需要注意，这个会话保持时间是最大无响应超时时间，即，用户在操作动态页面是，指定描述内无任何操作
											# 则接下来的操作会被分发到其他节点，若一直操作则不受该设置限制
	    protocol TCP						# 指定转发协议类型，有TCP和UDP
	
	    real_server 192.168.201.100 443 {	# 配置服务节点需要指定 real_server 的真实IP地址和端口，IP与端口之间用空格隔开
	        weight 1						# 配置服务节点的权值，权值大小用数字表示，数字越大，权值越高
											# 设置权值的大小可以为不同性能的服务器分配不同的负载，可以为性能搞的服务器设置较高的权值，
											# 性能低的设置相对低的权值
			TCP_CHECK {
				connect_timeout 3
				nb_get_retry 3
				delay_before_retry 3
			}								
			
	        SSL_GET {
	            url {
	              path /
	              digest ff20ad2481f97b1754ef3e12ecd3a9cc
	            }
	            url {
	              path /mrtg/
	              digest 9b3a0c85a887a256d6939da88aabd8cd
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	}
	
	virtual_server 10.10.10.2 1358 {
	    delay_loop 6
	    lb_algo rr
	    lb_kind NAT
	    persistence_timeout 50
	    protocol TCP
	
	    sorry_server 192.168.200.200 1358
	
	    real_server 192.168.200.2 1358 {
	        weight 1
	        HTTP_GET {
	            url {
	              path /testurl/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl2/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl3/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	
	    real_server 192.168.200.3 1358 {
	        weight 1
	        HTTP_GET {
	            url {
	              path /testurl/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334c
	            }
	            url {
	              path /testurl2/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334c
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	}
	
	virtual_server 10.10.10.3 1358 {
	    delay_loop 3
	    lb_algo rr
	    lb_kind NAT
	    nat_mask 255.255.255.0
	    persistence_timeout 50
	    protocol TCP
	
	    real_server 192.168.200.4 1358 {
	        weight 1
	        HTTP_GET {
	            url {
	              path /testurl/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl2/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl3/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	
	    real_server 192.168.200.5 1358 {
	        weight 1
	        HTTP_GET {
	            url {
	              path /testurl/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl2/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            url {
	              path /testurl3/test.jsp
	              digest 640205b7b0fc66c1ea91c463fac6334d
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	}
