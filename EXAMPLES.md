# 示例

真实代码示例，演示四大原则。每个示例展示LLM常见的错误做法以及如何修正。

---

## 1. 先思考，后编码

### 示例 1：隐藏的假设

**用户请求：** "添加一个导出用户数据的功能"

**❌ LLM的做法（错误假设）**

```python
def export_users(format='json'):
    """导出所有用户到文件。"""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"导出了 {len(users)} 个用户"
```

**问题：**
- 假设应该导出所有用户（那分页呢？隐私呢？）
- 假设了文件位置而没有询问
- 假设了要包含哪些字段
- 假设了CSV字段名而没有检查实际数据结构

**✅ 应该的做法（暴露假设）**

```
在实现之前，我需要澄清：

1. **范围**：导出所有用户还是筛选后的子集？（涉及隐私影响）
2. **格式**：你说的是"导出"——是指：
   - 浏览器中下载文件？
   - 后台任务发送邮件通知？
   - 返回数据的API端点？
3. **字段**：要包含哪些用户字段？（有些可能是敏感信息）
4. **数据量**：通常有多少用户？（影响实现方式）

最简单的方案：添加一个返回分页JSON的API端点。
如果需要文件导出，需要更多信息。

你倾向哪种方式？
```

### 示例 2：多种解读

**用户请求：** "让搜索更快"

**❌ LLM的做法（默默选择）**

```python
# 在没有询问哪种"更快"更重要的情况下
# 添加缓存、数据库索引和异步处理
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200行优化代码
```

**✅ 应该的做法**

```
"让搜索更快"可能意味着：

1. **更快的响应时间**（从500ms降到100ms以内）
   - 添加数据库索引
   - 缓存频繁查询
   - 预估工作量：2小时

2. **处理更多并发搜索**（吞吐量）
   - 添加async/await
   - 连接池
   - 预估工作量：4小时

3. **更快的感知速度**（用户体验）
   - 立即显示部分结果
   - 渐进式加载
   - 预估工作量：3小时

当前搜索典型查询耗时约500ms。
对你的使用场景来说，哪方面最重要？
```

---

## 2. 简洁至上

### 示例 1：过度抽象

**用户请求：** "添加一个计算折扣的函数"

**❌ LLM的做法（过度工程化）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 使用需要30多行来设置一个简单的计算
```

**✅ 应该的做法（简单）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """计算折扣金额。percent应为0-100。"""
    return amount * (percent / 100)

# 用法
discount = calculate_discount(100.0, 10.0)  # 减10美元
```

**何时添加复杂性：** 只有当你真的需要多种折扣类型时才添加。如果那是后面才有的需求，届时再重构。

### 示例 2：推测性功能

**用户请求：** "保存用户偏好到数据库"

**❌ LLM的做法（不必要的功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """保存偏好，支持可选的合并、验证和通知。"""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # 又30行代码来实现一个没人要的功能
        pass
```

**✅ 应该的做法（只做被要求的）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """保存用户偏好到数据库。"""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**后续按需添加：** 缓存（当性能成为问题时）、验证（当出现坏数据时）、合并（当需求出现时）。

---

## 3. 精准修改

### 示例 1：顺便重构

**用户请求：** "修复空邮箱导致验证器崩溃的bug"

**❌ LLM的做法（改动太多）**

```diff
  def validate_user(user_data):
-     # 检查邮箱格式
-     if not user_data.get('email'):
+     """验证用户数据。"""
+     email = user_data.get('email', '').strip()
+     
+     # 验证邮箱
+     if not email:
          raise ValueError("邮箱不能为空")
-     
-     # 基本邮箱验证
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("邮箱格式无效")
      
-     # 检查用户名
-     if not user_data.get('username'):
+     # 验证用户名
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("用户名不能为空")
+     if len(username) < 3:
+         raise ValueError("用户名太短")
+     if not username.isalnum():
+         raise ValueError("用户名必须为字母或数字")
      
      return True
```

**问题：**
- "改进了"超出bug修复范围的邮箱验证
- 添加了没人要求的用户名验证
- 修改了注释
- 添加了文档字符串

**✅ 应该的做法（精准手术）**

```diff
  def validate_user(user_data):
      # 检查邮箱格式
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("邮箱不能为空")
      
      # 基本邮箱验证
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("邮箱格式无效")
      
      # 检查用户名
      if not user_data.get('username'):
          raise ValueError("用户名不能为空")
      
      return True
```

**只改动了：** 修复空邮箱处理问题的具体代码行。

### 示例 2：风格漂移

**用户请求：** "给上传函数添加日志"

**❌ LLM的做法（重格式化一切）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """上传文件到目标地址，带日志记录。"""
+     logger.info(f"正在上传 {file_path} 到 {destination}")
+     
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         
-         response = requests.post(destination, files={'file': data})
-         
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+         
+         if success:
+             logger.info(f"上传成功: {file_path}")
+         else:
+             logger.error(f"上传失败: {file_path}, status={response.status_code}")
+         
+         return success
      except Exception as e:
-         print(f"错误: {e}")
+         logger.exception(f"上传错误 {file_path}: {e}")
          return False
```

**问题：**
- 改了引号风格（'' 改为 ""）
- 添加了没人要求的类型提示
- 添加了文档字符串
- 重格式化了空白
- 改了布尔返回逻辑

**✅ 应该的做法（匹配现有风格）**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'开始上传: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'上传成功: {file_path}')
              return True
          else:
+             logger.error(f'上传失败: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"错误: {e}")
+         logger.exception(f'上传错误: {file_path}')
          return False
```

**保持一致：** 单引号、无类型提示、现有布尔模式、间距风格。

---

## 4. 目标驱动执行

### 示例 1：模糊 vs 可验证

**用户请求：** "修复认证系统"

**❌ LLM的做法（模糊方法）**

```
我会通过以下方式修复认证系统：
1. 审查代码
2. 找出问题
3. 进行改进
4. 测试更改

[在没有明确成功标准的情况下进行修改]
```

**✅ 应该的做法（可验证的目标）**

```
要"修复认证"，我需要定义成功标准。
我们要解决的具体问题是什么？

如果问题是"改密码后用户仍然保持登录状态"：

计划：
1. 编写测试：修改密码 → 验证旧会话被取消
   验证：测试失败（复现bug）

2. 实现：密码修改时使会话失效
   验证：测试通过

3. 检查边界情况：多个活跃会话、并发修改
   验证：附加测试通过

4. 验证无回归：现有认证测试仍然通过
   验证：完整测试套件通过

当前认证测试覆盖率：[显示覆盖率]
你看到的具体认证问题是什么？
```

### 示例 2：多步骤与验证

**用户请求：** "给API添加速率限制"

**❌ LLM的做法（一次搞定）**

```python
# 一次实现完整的速率限制系统，包括Redis、多种策略、
# 配置系统和监控，共300行提交
# 没有明确的验证步骤
```

**✅ 应该的做法（增量验证）**

```
速率限制计划：

1. 添加基本的内存速率限制（单个端点）
   验证：
   - 测试：100个请求 → 前10个成功，其余得到429
   - 手动：curl该端点11次，看到速率限制错误

2. 提取为中间件（应用到所有端点）
   验证：
   - 测试：/users 和 /posts 都应用了速率限制
   - 现有端点测试仍然通过

3. 添加Redis后端（用于多服务器）
   验证：
   - 测试：速率限制在应用重启后保持
   - 测试：两个应用实例共享速率限制计数器

4. 添加配置（每个端点不同的速率）
   验证：
   - 测试：/search允许10次/分钟，/users允许100次/分钟
   - 配置文件正确解析

每一步都是独立可验证和可部署的。
从第1步开始？
```

### 示例 3：测试先行验证

**用户请求：** "当有重复分数时排序会出错"

**❌ LLM的做法（不复现就修复）**

```python
# 立即更改排序逻辑而不确认bug
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 应该的做法（先复现）**

```python
# 1. 首先，写一个复现问题的测试
def test_sort_with_duplicate_scores():
    """测试多个项目分数相同时的排序。"""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # Bug：重复项的顺序是不确定的
    # 运行这个测试多次，它应该是一致的
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：运行测试10次 → 因顺序不一致而失败

# 2. 现在用稳定排序修复
def sort_scores(scores):
    """按分数降序排序，分数相同时按名字升序。"""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试一致通过
```

---

## 反模式总结

| 原则 | 反模式 | 修复 |
|-----------|-------------|------|
| 先思考，后编码 | 默默假设文件格式、字段、范围 | 明确列出假设，寻求澄清 |
| 简洁至上 | 用策略模式处理单一折扣计算 | 用一个函数解决问题直到确实需要复杂性 |
| 精准修改 | 修bug时顺便改引号、添类型提示 | 只改能修复报告问题的代码行 |
| 目标驱动 | "我会审查并改进代码" | "为bug X写测试 → 让它通过 → 验证无回归" |

## 核心洞见

"过度复杂"的例子并非明显错误——它们遵循设计模式和最佳实践。问题在于**时机**：在需要之前就添加了复杂性，这会：

- 使代码更难理解
- 引入更多bug
- 花费更长时间实现
- 更难测试

"简单"的版本则：
- 更容易理解
- 实现更快
- 更容易测试
- 后续当复杂性真正需要时可以重构

**好的代码是能简单解决当下问题的代码，而不是提前解决未来问题的代码。**
