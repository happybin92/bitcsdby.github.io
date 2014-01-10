---
author: bydiao
comments: true
date: 2013-08-21 11:27:33+00:00
layout: post
slug: net-%e5%ae%9e%e7%8e%b0%e8%bf%9e%e6%8e%a5%e8%b6%85%e6%97%b6%e5%8f%af%e6%8e%a7%e7%9a%84tcpclient%e7%b1%bb
title: .Net 实现连接超时可控的TcpClient类
wordpress_id: 413
categories:
- .NET&amp;Windows编程
---

好久没写文章了，这一个月放了半个月假，整理了一下心情，为新的学年蓄积了一些力量。

现实中的工作还是无法逃避的，新版实验箱程序基本完成，说实话，还是很有成就感的。应该用一两篇文章整理一下自己的工作，整套程序涵盖了很多.net的初级中级甚至高级的知识。

今天来说说超时可控的TcpClient类的实现。

.net中默认的TcpClient类，在Server没有开启的时候会一直阻塞在那，这个时间在java中是可控的，而在.net中默认是不可控的，系统最多会阻塞60s的时间来连server，这就产生很大的延时，用户体验不好。必须要实时的发现Server没有开启，因此，通过网上的一些经典代码，实现了一个超时可控的TcpClient类如下：

[code lang="csharp"]
class TcpClientTimeout
    {

        private static bool IsConnectionSuccessful = false;

        private static Exception socketexception;

        private static ManualResetEvent TimeoutObject = new ManualResetEvent(false);

        /*
           外部可以调用次函数生成一个TcpClient
           TcpClient tcp = TcpClientTimeout.TcpClient(ipe,timeout);
        */
        public static TcpClient Connect(IPEndPoint remoteEndPoint, int timeoutMSec)
        {
            //将事件设置为非终止
            TimeoutObject.Reset();

            socketexception = null;

            string serverip = Convert.ToString(remoteEndPoint.Address);

            int serverport = remoteEndPoint.Port;

            TcpClient tcpclient = new TcpClient();


            //异步请求连接一个远程Server，回调函数是CallBackMethod
            tcpclient.BeginConnect(serverip, serverport,

                new AsyncCallback(CallBackMethod), tcpclient);

            //本函数开始阻塞等待timeoutMSec的时间，如果在时间内成功连接，会调用回调函数，进入if中执行
            //如果超过时间，会进入else执行，关闭现有连接，抛出一个超时异常
            if (TimeoutObject.WaitOne(timeoutMSec, false))
            {

                if (IsConnectionSuccessful)
                {
                    return tcpclient;
                }

                else
                {
                    throw socketexception;
                }
            }

            else
            {

                tcpclient.Close();

                throw new TimeoutException("TimeOut Exception");

            }

        }

        private static void CallBackMethod(IAsyncResult asyncresult)
        {

            try
            {

                IsConnectionSuccessful = false;

                TcpClient tcpclient = asyncresult.AsyncState as TcpClient;



                if (tcpclient.Client != null)
                {
                    //接受现有连接，标记连接已成功
                    tcpclient.EndConnect(asyncresult);

                    IsConnectionSuccessful = true;

                }

            }

            catch (Exception ex)
            {

                IsConnectionSuccessful = false;

                socketexception = ex;

            }

            finally
            {
                //设置事件为终止状态
                TimeoutObject.Set();

            }

        }

    }
[/code]

以上类主要用到了TcpClient的异步连接方式和 用于进程间通信的 ManualResetEvent 类，
最终实现超时可控的TcpClient类，解决了问题。
