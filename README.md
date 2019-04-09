# redlock
分布式锁几个注意点:
1, 锁的时间(防止程序中断，未释放锁，加过期时间)
2，释放锁(不能释放不是自己加的锁，加个随机值)
3，过期后释放锁应该出错
4，释放锁应该为原子操作(lua脚本)

https://www.cnblogs.com/rgcLOVEyaya/p/RGC_LOVE_YAYA_1003days.html
