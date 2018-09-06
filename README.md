# spring-data-redis-scan
在使用 spring data redis 时 遇到的问题与解答

在使用spring-boot集成redis时 遇到了redis连接池 在使用完后不及时释放 导致连接占满 无法获取redis连接的问题

经过调查发现是 使用 redisTemplate.opsForSet().scan() 方法时 在获取完数据后 并不会释放redis连接connection

解决的方法就是 重写DefaultSetOperations类的scan方法 代码如下：

private Cursor<Object> myScan(String key, final ScanOptions options) {
    return (Cursor<Object>) redisTemplate.executeWithStickyConnection((connection) -> {
        try {
            return new ConvertingCursor<>(connection.sScan(rawKey(key), options), source -> deserializeValue(source));
        } finally {
            RedisConnectionUtils.releaseConnection(connection, redisTemplate.getConnectionFactory()); // 使用完connection后 及时释放
        }
    });
}

这应该是【spring-boot-starter-data-redis】jar部分版本会遇到的问题 我的版本是【1.5.2.RELEASE】
