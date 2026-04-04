# AI Testing Guide for Omni-X Twitter Skills

## 如何在 AI 中测试这个 Skill

本指南提供了多种方式让 AI 代理测试和使用 Omni-X Twitter Skills。

---

## 方法 1: 直接运行示例代码（最简单）

### 步骤 1: 运行基础示例

```bash
# 运行 AI 代理示例
python agent_example.py
```

这将展示：
- 如何发现可用的技能
- 如何执行技能
- 如何处理响应和错误

### 步骤 2: 运行带认证的测试

```bash
# 运行带 token 的测试
python test_with_token.py
```

---

## 方法 2: 在 AI 代理中集成测试

### 2.1 基础集成测试

创建一个测试文件 `test_ai_integration.py`:

```python
from scripts import TwitterSkillInterface

def test_basic_integration():
    """测试基础集成功能"""
    print("=== AI Integration Test ===\n")
    
    # 初始化接口
    interface = TwitterSkillInterface()
    
    # 测试 1: 发现技能
    print("Test 1: Skill Discovery")
    skills = interface.get_available_skills()
    print(f"✓ Found {len(skills)} skills")
    print(f"  Skills: {list(skills.keys())}\n")
    
    # 测试 2: 执行无需登录的技能
    print("Test 2: Execute skill without login")
    result = interface.execute_skill(
        skill_name="get_user_profile",
        parameters={"username": "elonmusk"}
    )
    print(f"✓ Success: {result['success']}")
    print(f"  Skill: {result.get('skill_name')}")
    print(f"  Has data: {bool(result.get('data'))}\n")
    
    # 测试 3: 执行需要登录的技能（无 token）
    print("Test 3: Execute skill requiring login (no token)")
    result = interface.execute_skill(
        skill_name="get_user_followers",
        parameters={"username": "elonmusk", "count": 5}
    )
    print(f"✓ Success: {result['success']}")
    if not result['success']:
        print(f"  Expected error: {result.get('error')}\n")
    
    print("=== Test Complete ===")

if __name__ == "__main__":
    test_basic_integration()
```

运行测试：
```bash
python test_ai_integration.py
```

### 2.2 带认证的完整测试

创建 `test_ai_with_auth.py`:

```python
from scripts import TwitterSkillInterface
import os

def test_with_authentication():
    """测试带认证的完整功能"""
    print("=== AI Authentication Test ===\n")
    
    # 从环境变量获取 token（推荐方式）
    auth_token = os.getenv("TWITTER_AUTH_TOKEN", "your_token_here")
    
    # 初始化带 token 的接口
    interface = TwitterSkillInterface(auth_token=auth_token)
    
    # 测试所有技能
    test_cases = [
        {
            "name": "Get User Profile",
            "skill": "get_user_profile",
            "params": {"username": "elonmusk"}
        },
        {
            "name": "Get User Tweets",
            "skill": "get_user_tweets",
            "params": {"username": "elonmusk", "count": 3}
        },
        {
            "name": "Get User Followers",
            "skill": "get_user_followers",
            "params": {"username": "elonmusk", "count": 5}
        },
        {
            "name": "Get User Followings",
            "skill": "get_user_followings",
            "params": {"username": "elonmusk", "count": 5}
        },
        {
            "name": "Search Tweets",
            "skill": "search_tweets",
            "params": {"query": "AI", "count": 3, "search_filter": "Latest"}
        }
    ]
    
    for i, test in enumerate(test_cases, 1):
        print(f"Test {i}: {test['name']}")
        result = interface.execute_skill(
            skill_name=test['skill'],
            parameters=test['params']
        )
        
        if result['success']:
            print(f"  ✓ Success")
            print(f"    Count: {result.get('count', 'N/A')}")
            print(f"    Has next page: {result.get('has_next_page', 'N/A')}")
        else:
            print(f"  ✗ Failed: {result.get('error')}")
        print()
    
    print("=== Test Complete ===")

if __name__ == "__main__":
    test_with_authentication()
```

运行测试：
```bash
# 设置环境变量
export TWITTER_AUTH_TOKEN="your_actual_token"

# 运行测试
python test_ai_with_auth.py
```

---

## 方法 3: 在 AI 对话中测试

### 3.1 让 AI 代理执行测试

你可以直接让 AI 代理（如 Claude、GPT-4 等）执行以下代码：

```python
# AI 代理可以直接运行这段代码
from scripts import TwitterSkillInterface

# 初始化
interface = TwitterSkillInterface()

# 发现技能
skills = interface.get_available_skills()
print("Available skills:", list(skills.keys()))

# 测试一个技能
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 5}
)

print("Result:", result)
```

### 3.2 AI 代理测试脚本

创建一个 AI 友好的测试脚本 `ai_quick_test.py`:

```python
"""
Quick test script for AI agents
AI 代理快速测试脚本
"""

from scripts import TwitterSkillInterface
import json

def quick_test():
    """快速测试所有功能"""
    interface = TwitterSkillInterface()
    
    # 测试用例
    tests = {
        "profile": {
            "skill": "get_user_profile",
            "params": {"username": "elonmusk"},
            "requires_auth": False
        },
        "tweets": {
            "skill": "get_user_tweets",
            "params": {"username": "elonmusk", "count": 3},
            "requires_auth": False
        },
        "followers": {
            "skill": "get_user_followers",
            "params": {"username": "elonmusk", "count": 3},
            "requires_auth": True
        }
    }
    
    results = {}
    
    for test_name, test_config in tests.items():
        print(f"\nTesting: {test_name}")
        print(f"  Requires auth: {test_config['requires_auth']}")
        
        result = interface.execute_skill(
            skill_name=test_config['skill'],
            parameters=test_config['params']
        )
        
        results[test_name] = {
            "success": result['success'],
            "has_data": bool(result.get('data')),
            "count": result.get('count', 0),
            "error": result.get('error')
        }
        
        print(f"  Result: {'✓ Success' if result['success'] else '✗ Failed'}")
        if result.get('error'):
            print(f"  Error: {result['error']}")
    
    # 输出总结
    print("\n" + "="*50)
    print("Test Summary:")
    print(json.dumps(results, indent=2))
    
    return results

if __name__ == "__main__":
    quick_test()
```

---

## 方法 4: 使用 Python REPL 交互测试

在终端中启动 Python 交互式环境：

```bash
python
```

然后逐步测试：

```python
# 导入模块
from scripts import TwitterSkillInterface

# 初始化
interface = TwitterSkillInterface()

# 查看可用技能
skills = interface.get_available_skills()
print(list(skills.keys()))

# 查看某个技能的详细信息
print(skills['get_user_tweets'])

# 执行技能
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 5}
)

# 查看结果
print(result['success'])
print(result.get('count'))
print(result.get('data')[:2] if result.get('data') else None)
```

---

## 方法 5: 在 Jupyter Notebook 中测试

创建一个 Notebook `test_omni_x.ipynb`:

```python
# Cell 1: 导入和初始化
from scripts import TwitterSkillInterface
import json

interface = TwitterSkillInterface()
print("✓ Interface initialized")

# Cell 2: 发现技能
skills = interface.get_available_skills()
print(f"Found {len(skills)} skills:")
for name in skills.keys():
    print(f"  - {name}")

# Cell 3: 测试获取用户资料
result = interface.execute_skill(
    skill_name="get_user_profile",
    parameters={"username": "elonmusk"}
)
print(json.dumps(result, indent=2, default=str))

# Cell 4: 测试获取推文
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 5}
)
print(f"Success: {result['success']}")
print(f"Count: {result.get('count')}")

# Cell 5: 测试错误处理
result = interface.execute_skill(
    skill_name="invalid_skill",
    parameters={}
)
print(f"Error handled: {not result['success']}")
print(f"Error message: {result.get('error')}")
```

---

## 获取 Auth Token 的方法

如果需要测试需要认证的功能，按以下步骤获取 token：

### 方法 A: 通过浏览器开发者工具

1. 在浏览器中登录 Twitter/X
2. 按 F12 打开开发者工具
3. 转到 **Application** 标签（Chrome）或 **Storage** 标签（Firefox）
4. 展开 **Cookies** → 点击 `https://twitter.com` 或 `https://x.com`
5. 找到名为 `auth_token` 的 cookie
6. 复制其 **Value** 值

### 方法 B: 通过 Network 标签

1. 打开开发者工具（F12）→ **Network** 标签
2. 刷新页面
3. 点击任意 Twitter API 请求
4. 查看 **Headers** → **Request Headers**
5. 找到 `Cookie:` 头部
6. 在 cookie 字符串中找到 `auth_token=...`
7. 复制 `auth_token=` 后面的值

### 使用 Token

```python
# 方法 1: 直接传入
interface = TwitterSkillInterface(auth_token="your_token_here")

# 方法 2: 从环境变量
import os
token = os.getenv("TWITTER_AUTH_TOKEN")
interface = TwitterSkillInterface(auth_token=token)

# 方法 3: 后续设置
interface = TwitterSkillInterface()
interface.set_auth_token("your_token_here")
```

---

## 常见测试场景

### 场景 1: 测试基础功能（无需登录）

```python
from scripts import TwitterSkillInterface

interface = TwitterSkillInterface()

# 获取用户资料
profile = interface.execute_skill(
    skill_name="get_user_profile",
    parameters={"username": "elonmusk"}
)

# 获取推文
tweets = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 10}
)

print(f"Profile success: {profile['success']}")
print(f"Tweets success: {tweets['success']}")
print(f"Tweets count: {tweets.get('count')}")
```

### 场景 2: 测试完整功能（需要登录）

```python
from scripts import TwitterSkillInterface

interface = TwitterSkillInterface(auth_token="your_token")

# 获取关注者
followers = interface.execute_skill(
    skill_name="get_user_followers",
    parameters={"username": "elonmusk", "count": 20}
)

# 搜索推文
search = interface.execute_skill(
    skill_name="search_tweets",
    parameters={"query": "AI technology", "count": 10, "search_filter": "Latest"}
)

print(f"Followers: {followers.get('count')}")
print(f"Search results: {search.get('count')}")
```

### 场景 3: 测试错误处理

```python
from scripts import TwitterSkillInterface

interface = TwitterSkillInterface()

# 测试无效技能
result = interface.execute_skill(
    skill_name="invalid_skill",
    parameters={}
)

print(f"Handled error: {not result['success']}")
print(f"Error message: {result.get('error')}")
print(f"Available skills: {result.get('available_skills')}")
```

---

## AI 代理集成最佳实践

### 1. 动态技能发现

```python
# AI 应该首先发现可用技能
skills = interface.get_available_skills()

# 然后根据用户需求选择合适的技能
if "get profile" in user_request.lower():
    skill_name = "get_user_profile"
elif "get tweets" in user_request.lower():
    skill_name = "get_user_tweets"
```

### 2. 错误处理

```python
result = interface.execute_skill(skill_name, parameters)

if result['success']:
    # 处理成功的结果
    data = result['data']
    process_data(data)
else:
    # 处理错误
    error_message = result['error']
    handle_error(error_message)
```

### 3. 参数验证

```python
# 在执行前验证参数
skill_info = skills[skill_name]
required_params = skill_info['parameters']

# 确保所有必需参数都存在
for param in required_params:
    if param['required'] and param['name'] not in parameters:
        print(f"Missing required parameter: {param['name']}")
```

---

## 故障排除

### 问题 1: 导入错误

```bash
# 确保已安装依赖
pip install -r requirements.txt

# 确保在正确的目录
cd /path/to/Omni-X
python test.py
```

### 问题 2: 认证失败

```python
# 检查 token 是否有效
interface = TwitterSkillInterface(auth_token="your_token")

# 测试需要认证的功能
result = interface.execute_skill(
    skill_name="get_user_followers",
    parameters={"username": "elonmusk", "count": 5}
)

if not result['success']:
    print(f"Auth error: {result['error']}")
    # 可能需要重新获取 token
```

### 问题 3: 速率限制

```python
import time

# 在请求之间添加延迟
for username in usernames:
    result = interface.execute_skill(
        skill_name="get_user_profile",
        parameters={"username": username}
    )
    time.sleep(2)  # 等待 2 秒
```

---

## 总结

选择最适合你的测试方法：

1. **快速测试**: 运行 `python agent_example.py`
2. **完整测试**: 创建自定义测试脚本
3. **交互测试**: 使用 Python REPL 或 Jupyter Notebook
4. **AI 集成**: 在 AI 代理中直接使用接口

所有方法都使用相同的 `TwitterSkillInterface`，确保一致的体验。
