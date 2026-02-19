---
name: workspace-archiver-v2
description: Archive and backup OpenClaw workspace with versioning and compression. Use when backing up workspace, archiving old projects, creating snapshots, or managing workspace storage. Triggers on "archive workspace", "backup", "workspace snapshot", "archive project".
---

# Workspace Archiver V2

**Version:** 2.0.0 | **Author:** OpenClaw

OpenClaw 工作空间归档和备份工具，支持版本控制、压缩和自动清理。

## 功能特性

- 📦 **智能归档** - 自动识别并归档已完成项目
- 🗜️ **多种压缩** - 支持 tar.gz, zip, 7z 等格式
- 🔄 **版本控制** - 保留多个历史版本
- 📊 **存储分析** - 分析工作空间存储使用情况
- 🧹 **自动清理** - 自动删除过期归档
- 🔐 **加密支持** - 可选密码保护归档文件

## 安装

```bash
npm install workspace-archiver-v2
```

## 使用方法

### CLI 使用

```bash
# 归档整个工作空间
workspace-archiver archive --all

# 归档特定项目
workspace-archiver archive ./projects/my-project

# 创建带版本的快照
workspace-archiver snapshot --name "pre-release-v2"

# 恢复归档
workspace-archiver restore archive-2026-02-19.tar.gz --to ./restored

# 查看存储分析
workspace-archiver analyze

# 清理过期归档
workspace-archiver cleanup --keep 10 --older-than 30d
```

### 作为库使用

```javascript
const { WorkspaceArchiver } = require('workspace-archiver-v2');

const archiver = new WorkspaceArchiver({
  workspaceRoot: '/path/to/workspace',
  archiveDir: '/path/to/archives',
  compression: 'gzip',
  encrypt: false
});

// 归档项目
await archiver.archive('./my-project', {
  name: 'my-project-backup',
  version: 'v1.0.0',
  exclude: ['node_modules', '.git', '*.log']
});

// 创建快照
await archiver.snapshot({
  name: 'daily-backup',
  include: ['projects', 'docs'],
  compress: true
});

// 恢复归档
await archiver.restore('my-project-backup-v1.0.0.tar.gz', {
  to: './restored',
  overwrite: false
});
```

## 示例

### 示例 1: 基础归档

```bash
# 归档当前项目
$ workspace-archiver archive .

📦 Archiving workspace...

Analyzing: ./my-project
  Size: 15.3 MB
  Files: 1,247
  Excluding: node_modules/, .git/, *.log

Compressing: gzip
  Progress: 100% [████████████████████]

✅ Archive created:
   File: my-project-2026-02-19.tar.gz
   Size: 4.2 MB (72% reduction)
   Location: /archives/my-project-2026-02-19.tar.gz
   Checksum: sha256:a1b2c3...
```

### 示例 2: 自动归档策略

```javascript
const { WorkspaceArchiver, AutoArchivePolicy } = require('workspace-archiver-v2');

const archiver = new WorkspaceArchiver();

// 配置自动归档策略
const policy = new AutoArchivePolicy({
  // 归档条件
  conditions: {
    // 超过 30 天未修改的项目
    lastModified: '30d',
    // 标记为完成的项目
    hasTag: 'completed',
    // 超过 100MB 的项目
    sizeThreshold: '100MB'
  },
  // 归档选项
  options: {
    compress: true,
    encrypt: false,
    keepVersions: 5
  },
  // 清理策略
  cleanup: {
    keepLast: 10,
    olderThan: '90d'
  }
});

// 执行自动归档
const result = await archiver.autoArchive(policy);

console.log(`Archived ${result.archived.length} projects`);
console.log(`Freed ${result.spaceFreed} MB`);
```

### 示例 3: 定时备份脚本

```bash
#!/bin/bash
# backup-daily.sh

BACKUP_NAME="workspace-$(date +%Y-%m-%d)"
ARCHIVE_DIR="/backups/workspace"
RETENTION_DAYS=30

echo "Starting daily backup..."

# 创建快照
workspace-archiver snapshot \
  --name "$BACKUP_NAME" \
  --output "$ARCHIVE_DIR" \
  --compress gzip \
  --exclude node_modules,.git,*.tmp

# 验证备份
if [ $? -eq 0 ]; then
  echo "✅ Backup completed: $ARCHIVE_DIR/$BACKUP_NAME.tar.gz"
  
  # 清理旧备份
  echo "Cleaning up backups older than $RETENTION_DAYS days..."
  find "$ARCHIVE_DIR" -name "workspace-*.tar.gz" -mtime +$RETENTION_DAYS -delete
  
  echo "✅ Cleanup completed"
else
  echo "❌ Backup failed"
  exit 1
fi

# 添加到 crontab (每天凌晨 2 点)
# 0 2 * * * /path/to/backup-daily.sh >> /var/log/workspace-backup.log 2>&1
```

### 示例 4: 存储分析

```bash
$ workspace-archiver analyze

📊 Workspace Storage Analysis
==============================

Total Size: 2.3 GB
Total Files: 45,678

By Category:
  Projects:     1.8 GB (78%)
  Archives:     320 MB (14%)
  Logs:          95 MB (4%)
  Temp:          45 MB (2%)
  Other:         40 MB (2%)

Largest Projects:
  1. project-a:      450 MB (last modified: 2d ago)
  2. project-b:      380 MB (last modified: 15d ago)
  3. project-c:      290 MB (last modified: 45d ago)

Recommendations:
  ⚠️  project-b: Consider archiving (15d inactive)
  ⚠️  project-c: Should be archived (45d inactive)
  💡 Logs: Can save 80MB with cleanup
```

## 配置选项

### 全局配置

```javascript
// archiver.config.js
module.exports = {
  // 工作空间根目录
  workspaceRoot: process.env.WORKSPACE_ROOT || './',
  
  // 归档存储目录
  archiveDir: './archives',
  
  // 默认压缩格式
  compression: 'gzip', // 'gzip' | 'bzip2' | 'xz' | 'zip' | '7z'
  
  // 压缩级别
  compressionLevel: 6, // 1-9
  
  // 默认排除模式
  exclude: [
    'node_modules',
    '.git',
    '*.log',
    '*.tmp',
    '.env',
    '.DS_Store',
    'Thumbs.db'
  ],
  
  // 加密设置
  encryption: {
    enabled: false,
    algorithm: 'aes-256-gcm'
  },
  
  // 版本控制
  versioning: {
    enabled: true,
    maxVersions: 10,
    naming: '{name}-v{version}-{date}'
  },
  
  // 自动清理
  cleanup: {
    enabled: true,
    keepLast: 10,
    olderThan: '90d'
  }
};
```

## API 参考

### WorkspaceArchiver

```javascript
const archiver = new WorkspaceArchiver(options);
```

**方法：**

```javascript
// 归档项目
archiver.archive(source, options): Promise<ArchiveResult>

// 创建快照
archiver.snapshot(options): Promise<ArchiveResult>

// 恢复归档
archiver.restore(archivePath, options): Promise<RestoreResult>

// 列出归档
archiver.list(options): Promise<Archive[]>

// 删除归档
archiver.delete(archiveName): Promise<void>

// 分析存储
archiver.analyze(): Promise<AnalysisResult>

// 清理过期归档
archiver.cleanup(policy): Promise<CleanupResult>

// 自动归档
archiver.autoArchive(policy): Promise<AutoArchiveResult>
```

### ArchiveOptions

```typescript
interface ArchiveOptions {
  name?: string;           // 归档名称
  version?: string;        // 版本号
  compression?: string;    // 压缩格式
  level?: number;          // 压缩级别 1-9
  encrypt?: boolean;       // 是否加密
  password?: string;       // 加密密码
  exclude?: string[];      // 排除模式
  include?: string[];      // 包含模式（优先于 exclude）
  metadata?: object;       // 自定义元数据
}
```

### RestoreOptions

```typescript
interface RestoreOptions {
  to: string;              // 恢复目标目录
  overwrite?: boolean;     // 是否覆盖现有文件
  filter?: string[];       // 仅恢复匹配的文件
  verify?: boolean;        // 验证校验和
}
```

## 支持的压缩格式

| 格式 | 扩展名 | 压缩比 | 速度 | 说明 |
|------|--------|--------|------|------|
| gzip | .tar.gz | 中 | 快 | 默认格式，兼容性好 |
| bzip2 | .tar.bz2 | 高 | 中 | 更好的压缩比 |
| xz | .tar.xz | 最高 | 慢 | 最大压缩 |
| zip | .zip | 中 | 快 | Windows 兼容 |
| 7z | .7z | 高 | 中 | 需要额外安装 |

## 测试

```bash
# 运行单元测试
npm test

# 运行集成测试（实际文件操作）
npm run test:integration

# 带覆盖率报告
npm run test:coverage
```

## 与 CI/CD 集成

```yaml
# .github/workflows/backup.yml
name: Workspace Backup
on:
  schedule:
    - cron: '0 2 * * 0'  # 每周日凌晨 2 点
jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install archiver
        run: npm install -g workspace-archiver-v2
      - name: Create backup
        run: workspace-archiver snapshot --name "ci-backup"
      - name: Upload to S3
        run: aws s3 cp ./archives/ci-backup.tar.gz s3://backups/workspace/
```

## License

MIT
