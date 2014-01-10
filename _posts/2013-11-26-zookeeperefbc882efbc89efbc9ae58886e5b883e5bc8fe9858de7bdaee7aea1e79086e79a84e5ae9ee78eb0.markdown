---
author: bydiao
comments: true
date: 2013-11-26 13:06:56+00:00
layout: post
slug: zookeeper%ef%bc%882%ef%bc%89%ef%bc%9a%e5%88%86%e5%b8%83%e5%bc%8f%e9%85%8d%e7%bd%ae%e7%ae%a1%e7%90%86%e7%9a%84%e5%ae%9e%e7%8e%b0
title: Zookeeper（2）：分布式配置管理的实现
wordpress_id: 578
categories:
- Hadoop
---

Zookeeper为分布式系统提供协调统筹服务，其中最典型的便是分布式配置管理。
简单的说，有100个节点，现在需要更新config文件，用scp来实现文件远程拷贝显然不太现实，这就需要Zookeeper大显身手了。

整个分布式配置管理有三个角色：
配置修改发起者 ConfigUpdater
Zookeeper服务器集群
ZookeeperClient

基本流程如下，ZookeeperClient监听Zookeeper某一个ZNode下的值，ConfigUpdater修改Zookeeper某一个ZNode下的值。该ZNode的值一旦发生改变，则根据改变的类型，将该值的变化同步到本地。这里的值，就是Config文件。

下面是GitHub上一个分布式配置管理的简单实现，主要有两个模块，四个类
[code lang="java"]

//ConfigUpdater 模块的实现
import java.io.IOException;
import java.util.Random;
import java.util.concurrent.TimeUnit;

import org.apache.zookeeper.KeeperException;

public class ConfigUpdater {

    public static final String PATH = "/config";

    private ActiveKeyValueStore _store;
    private Random _random = new Random();

    public ConfigUpdater(String hosts) throws IOException, InterruptedException {
        _store = new ActiveKeyValueStore();
        _store.connect(hosts);//连接远程zkServer
    }

    public void run() throws InterruptedException, KeeperException {
        //noinspection InfiniteLoopStatement
        while (true) {
            String value = _random.nextInt(100) + "";
            _store.write(PATH, value);
            System.out.printf("Set %s to %s\n", PATH, value);
            TimeUnit.SECONDS.sleep(_random.nextInt(10));
        }//定期修改ZNode下的值
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
       // ConfigUpdater updater = new ConfigUpdater(args[0]);
    	ConfigUpdater updater = new ConfigUpdater("192.168.0.2");
        updater.run();
    }

}
[/code]

[code lang="java"]
//ConfigWatcher的实现
import java.io.IOException;

import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;

public class ConfigWatcher implements Watcher {

    private ActiveKeyValueStore _store;
    public static final String PATH = "/config";
    
    public ConfigWatcher(String hosts) throws InterruptedException, IOException {
        _store = new ActiveKeyValueStore();
        _store.connect(hosts);//连接远程zkServer
    }

    public void displayConfig() throws InterruptedException, KeeperException {
        String value = _store.read(PATH, this);
        System.out.printf("Read %s as %s\n", PATH, value);
    }//获取并显示ZNode下的值


    @Override//zkServer发生改变后触发此函数
    public void process(WatchedEvent event) {
        System.out.printf("Process incoming event: %s\n", event.toString());
        if (event.getType() == Event.EventType.NodeDataChanged) {
            try {
                displayConfig();
            }
            catch (InterruptedException e) {
                System.err.println("Interrupted. Exiting");
                Thread.currentThread().interrupt();
            }
            catch (KeeperException e) {
                System.err.printf("KeeperException: %s. Exiting.\n", e);
            }
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        //ConfigWatcher watcher = new ConfigWatcher(args[0]);
    	ConfigWatcher watcher = new ConfigWatcher("192.168.0.2");
        watcher.displayConfig();
        Thread.sleep(Long.MAX_VALUE);
    }

}
[/code]

[code lang="java"]
import java.nio.charset.Charset;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.data.Stat;


public class ActiveKeyValueStore extends ConnectionWatcher {

    private static final Charset CHARSET = Charset.forName("UTF-8");

    public void write(String path, String value) throws InterruptedException, KeeperException {
        Stat stat = zk.exists(path, false);
        if (stat == null) {//创建或者更新ZNode节点数据
            zk.create(path, value.getBytes(CHARSET), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        else {
            zk.setData(path, value.getBytes(CHARSET), -1);
        }
    }

    public String read(String path, Watcher watcher) throws InterruptedException, KeeperException {
        byte[] data = zk.getData(path, watcher, null /* stat */);
        return new String(data, CHARSET);
    }//读取ZNode数据


}
[/code]

[code lang="java"]
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

public class ConnectionWatcher implements Watcher {
    
    private static final int SESSION_TIMEOUT = 5000;
    protected ZooKeeper zk;
    private CountDownLatch _connectedSignal = new CountDownLatch(1);

    public void connect(String hosts) throws IOException, InterruptedException {
        zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
        _connectedSignal.await();
    }

    @Override
    public void process(WatchedEvent event) {
        if (event.getState() == Event.KeeperState.SyncConnected) {
            _connectedSignal.countDown();
        }
    }

    public void close() throws InterruptedException {
        zk.close();
    }
}
[/code]

两个模块连接到zkServer上，就可以简单模拟出一个配置管理服务了。
