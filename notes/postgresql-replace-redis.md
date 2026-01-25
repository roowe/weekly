# 原文总结：用 PostgreSQL 代替 Redis

**作者：** polliog (原文作者)
**原文标题：** I Replaced Redis With PostgreSQL and It's Faster
**原文链接：** https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942
**出处：** 科技爱好者周刊第381期
**总结日期：** 2026-01-25
**总结者：** Claude Code

---

## 原文摘要

作者描述了一个典型的Web应用栈：PostgreSQL用于持久化数据，Redis用于缓存、发布订阅和后台任务。但他意识到一个问题：两个数据库，两套管理，两个可能的故障点。

## 核心发现

PostgreSQL可以完成Redis的所有功能：
1. **缓存** - 使用UNLOGGED表格（不写入WAL日志，提高性能）
2. **发布订阅** - 使用LISTEN/NOTIFY（PostgreSQL内置的消息机制）
3. **任务队列** - 使用SKIP LOCKED（跳过被锁定的行，避免阻塞）
4. **会话管理** - 使用JSONB表格（存储结构化会话数据）

## 技术实现

### 1. 缓存功能
- 使用UNLOGGED表格替代Redis缓存
- 性能对比：Redis SET 0.05ms vs Postgres 0.08ms
- 通过表格+过期时间索引实现

### 2. 发布订阅功能
- 使用LISTEN/NOTIFY替代Redis pub/sub
- 性能对比：Redis 1-2ms vs Postgres 2-5ms
- 可结合触发器实现原子操作

### 3. 任务队列功能
- 使用FOR UPDATE SKIP LOCKED替代Redis队列
- 性能对比：Redis 0.1ms vs Postgres 0.3ms
- 支持多工作者并发处理

## 性能对比

### 单独操作速度
- **缓存写入**：Redis 0.05ms，Postgres 0.08ms（+60%）
- **缓存读取**：Redis 0.04ms，Postgres 0.06ms（+50%）
- **发布订阅**：Redis 1.2ms，Postgres 3.1ms（+158%）
- **队列入队**：Redis 0.08ms，Postgres 0.15ms（+87%）
- **队列出队**：Redis 0.12ms，Postgres 0.31ms（+158%）

### 组合操作优势
**场景**：插入数据 + 清理缓存 + 通知订阅者
- **Redis方案**：约4ms（包括网络跳转）
- **Postgres方案**：约2.2ms（同一连接，事务保证）

## 成本效益

**月度成本对比**：
- AWS ElastiCache 2GB：$45
- PostgreSQL RDS 20GB：$50（已在使用）
- 5GB额外存储：$0.50
- **潜在月节省**：~$100

## 适用场景建议

### 适合替换的情况：
1. 小型到中型应用
2. 简单的缓存需求（键值存储）
3. 需要事务一致性保证
4. 团队运营资源有限

### 应该保留Redis的情况：
1. 需要极高性能（10万+操作/秒）
2. 使用Redis特有数据结构（排序集合、HyperLogLog等）
3. 有专门的运维团队
4. 亚毫秒级延迟是关键



---

## 个人思考与疑问

### 值得思考的观点
1. **架构简化**：使用单一数据库确实可以简化架构，减少运维复杂度
2. **事务一致性**：PostgreSQL方案可以利用事务保证数据一致性，这是Redis无法提供的
3. **组合操作优势**：在涉及多个操作时，PostgreSQL的同一连接优势明显（2.2ms vs 4ms）

### 技术疑问
1. **并发处理能力**：PostgreSQL在高并发场景下的性能表现如何？
2. **内存管理**：Redis作为内存数据库的内存管理机制是否在PostgreSQL中能得到类似效果？
3. **数据过期清理**：PostgreSQL中如何高效实现类似于Redis的自动过期机制？

### 适用性评估
- **适合项目**：中小型项目、对一致性要求高、团队资源有限
- **不适合场景**：需要极高性能（10万+ QPS）、使用Redis特有数据结构、延迟敏感型应用

## 行动计划

### 学习方向
1. 深入学习PostgreSQL的UNLOGGED表格特性
2. 研究LISTEN/NOTIFY的并发处理机制
3. 实践SKIP LOCKED在任务队列中的应用

### 实践建议
1. 在下一个中小型项目中尝试使用PostgreSQL替代Redis
2. 建立性能测试基准，对比两种方案的实际情况
3. 总结实际应用中的经验教训

---
**总结完成时间：** 2026-01-25
**后续更新：** [在此添加后续的学习或实践心得]