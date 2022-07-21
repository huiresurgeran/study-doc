



雪花算法

https://segmentfault.com/a/1190000040964518



关键代码

```java
		// 序列号掩码 4095 (0b111111111111=0xfff=4095)
    // 用于序号的与运算，保证序号最大值在0-4095之间
    private long sequenceMask = -1L ^ (-1L << sequenceBits);
		
		// 获取下一个随机的ID
    public synchronized long nextId() {
        // 获取当前时间戳，单位毫秒
        long timestamp = timeGen();

        if (timestamp < lastTimestamp) {
            System.err.printf("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp);
            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds",
                    lastTimestamp - timestamp));
        }

        // 去重
        if (lastTimestamp == timestamp) {

            sequence = (sequence + 1) & sequenceMask;

            // sequence序列大于4095
            if (sequence == 0) {
                // 调用到下一个时间戳的方法
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            // 如果是当前时间的第一次获取，那么就置为0
            sequence = 0;
        }

        // 记录上一次的时间戳
        lastTimestamp = timestamp;

        // 偏移计算
        return ((timestamp - twepoch) << timestampLeftShift) |
                (datacenterId << datacenterIdShift) |
                (workerId << workerIdShift) |
                sequence;
    }

    private long tilNextMillis(long lastTimestamp) {
        // 获取最新时间戳
        long timestamp = timeGen();
        // 如果发现最新的时间戳小于或者等于序列号已经超4095的那个时间戳
        while (timestamp <= lastTimestamp) {
            // 不符合则继续
            timestamp = timeGen();
        }
        return timestamp;
    }

    private long timeGen() {
        return System.currentTimeMillis();
    }
```