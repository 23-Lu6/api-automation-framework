# 数据驱动接口自动化测试框架项目架构介绍

## 一、项目介绍

基于**Python+pytest+requests+allure+jsonpath+yaml+Jenkins**

该项目是一个企业级的接口自动化测试框架，采用数据驱动测试（DDT）设计理念，将测试数据与测试逻辑完全分离。测试人员只需编写YAML文件描述接口信息，框架会自动执行HTTP请求、动态参数替换、接口关联、断言验证，并生成可视化的Allure测试报告。框架支持多种断言方式（包含断言、相等断言、数据库断言等），可广泛应用于各类Web API接口的功能测试、回归测试和业务场景测试。

## 二、项目结构说明

```
project-root/                          # 项目根目录

├─ base/                               # 基础类封装（核心引擎）
│  ├─ apiutil.py                       # 接口请求处理基类（RequestBase核心类）
│  ├─ apiutil_business.py              # 业务场景处理类（多接口串联）
│  ├─ generateId.py                    # Allure报告ID生成器
│  ├─ new_testcase_tools.py            # YAML用例生成工具
│  ├─ new_tools.ui                     # GUI界面文件
│  └─ removefile.py                    # 文件清理工具

├─ common/                             # 公共方法封装（工具库）
│  ├─ sendrequest.py                   # HTTP请求发送封装（SendRequest类）
│  ├─ readyaml.py                      # YAML数据读写工具（ReadYamlData类）
│  ├─ debugtalk.py                     # 动态函数库（参数替换核心）
│  ├─ assertions.py                    # 断言逻辑封装（Assertions类）
│  ├─ connection.py                    # 数据库连接封装（MySQL/Redis/MongoDB等）
│  ├─ recordlog.py                     # 日志记录工具
│  ├─ operationcsv.py                  # CSV文件操作工具
│  ├─ handleExcel.py                   # Excel文件处理工具
│  ├─ operxml.py                       # XML文件操作工具
│  ├─ dingRobot.py                     # 钉钉机器人通知
│  ─ semail.py                        # 邮件发送工具

├─ conf/                               # 全局配置目录
│  ├─ config.ini                       # 环境配置文件（API地址、数据库等）
│  ├─ setting.py                       # 全局常量定义
│  └─ operationConfig.py               # 配置文件读取类

├─ data/                               # 测试数据目录
│  ├─ vehicleNo.csv                    # 车牌号测试数据
│  ├─ login_data.csv                   # 登录账号数据
│  ├─ loginName.yaml                   # 登录名配置
│  ├─ 测试数据.xls                      # Excel测试数据
│  └─ sql/                             # SQL语句仓库
│     ├─ newVehicleAddShare.xml        # 车辆新增SQL
│     └─ homePage.xml                  # 首页数据SQL

├─ logs/                               # 测试日志目录（自动生成）
│  ─ test.20261021.log                # 按日期命名的日志文件

├─ report/                             # 测试报告目录
│  ├─ temp/                            # Allure原始数据（自动生成）
│  ├─ allureReport/                    # Allure HTML报告（自动生成）
│  └─ tmreport/                        # TMReport报告（自动生成）

├─ testcase/                           # 测试用例目录
│  ├─ Single interface/                # 单接口测试用例
│  │  ├─ test_debug_api.py             # 用户管理测试类
│  │  ├─ addUser.yaml                  # 新增用户测试数据
│  │  ├─ updateUser.yaml               # 修改用户测试数据
│  │  ├─ deleteUser.yaml               # 删除用户测试数据
│  │  └─ queryUser.yaml                # 查询用户测试数据
│  ├─ Business interface/              # 业务场景测试用例
│  │  ├─ test_business_scenario.py     # 业务场景测试类
│  │  └─ BusinessScenario.yml          # 业务场景测试数据
│  └─ ProductManager/                  # 产品管理模块测试
│     ├─ test_productList.py           # 产品管理测试类
│     ├─ login_dw.yaml                 # 登录接口测试数据
│     ├─ getProductList.yaml           # 产品列表测试数据
│     ├─ productDetail.yaml            # 产品详情测试数据
│     ├─ commitOrder.yaml              # 提交订单测试数据
│     └─ orderPay.yaml                 # 订单支付测试数据

├─ venv/                               # 虚拟环境目录（自动生成）

├─ conftest.py                         # pytest全局钩子文件（固定名称）
├─ environment.xml                     # Allure报告环境信息文件
├─ extract.yaml                        # 接口依赖参数存储文件
─ pytest.ini                          # pytest配置文件（固定名称）
├─ requirements.txt                    # 第三方库依赖清单
└─ run.py                              # 主程序入口（执行测试和生成报告）
```

## 三、核心代码

### 1. 程序入口 run.py

#### 代码如下：

```python
import shutil
import pytest
import os
import webbrowser
from conf.setting import REPORT_TYPE

if __name__ == '__main__':
    if REPORT_TYPE == 'allure':
        pytest.main(
            ['-s', '-v', '--alluredir=./report/temp', './testcase', '--clean-alluredir',
             '--junitxml=./report/results.xml'])
        shutil.copy('./environment.xml', './report/temp')
        os.system(f'allure serve ./report/temp')
    elif REPORT_TYPE == 'tm':
        pytest.main(['-vs', '--pytest-tmreport-name=testReport.html', '--pytest-tmreport-path=./report/tmreport'])
        webbrowser.open_new_tab(os.getcwd() + '/report/tmreport/testReport.html')
```

#### 代码说明：

##### 主程序入口

```python
if __name__ == '__main__':
```

- Python标准写法，表示当脚本作为主程序运行时才执行以下代码块。
- 防止模块被其他脚本导入时意外执行测试逻辑。

##### Allure报告处理逻辑

```python
if REPORT_TYPE == 'allure':
    pytest.main(
        ['-s', '-v', '--alluredir=./report/temp', './testcase', '--clean-alluredir',
         '--junitxml=./report/results.xml'])
```

###### 参数说明：

| 参数 | 含义 |
|---|---|
| `-s` | 输出所有打印信息（不屏蔽stdout） |
| `-v` | 显示详细测试结果 |
| `--alluredir=./report/temp` | 将Allure报告数据保存到指定目录 |
| `./testcase` | 指定测试用例所在目录 |
| `--clean-alluredir` | 在每次运行前清空之前的报告数据 |
| `--junitxml=./report/results.xml` | 生成JUnit XML格式的测试结果文件 |

```python
shutil.copy('./environment.xml', './report/temp')
```

- 复制环境信息文件`environment.xml`到报告目录中。
- Allure报告会读取此文件并显示当前测试环境信息。

```python
os.system(f'allure serve ./report/temp')
```

- 使用Allure CLI命令启动本地服务器并展示生成的报告页面。
- 会在默认浏览器中自动打开报告。

### 2. 测试用例 testcase

#### 用户管理接口测试代码示例：

```python
import allure
import pytest
from common.readyaml import get_testcase_yaml
from base.apiutil import RequestBase
from base.generateId import m_id, c_id

@allure.feature(next(m_id) + '用户管理模块（单接口）')
class TestUserManager:
    @allure.story(next(c_id) + "新增用户")
    @pytest.mark.run(order=1)
    @pytest.mark.parametrize('base_info,testcase', get_testcase_yaml("./testcase/Single interface/addUser.yaml"))
    def test_add_user(self, base_info, testcase):
        allure.dynamic.title(testcase['case_name'])
        RequestBase().specification_yaml(base_info, testcase)
```

#### 代码说明：

##### `@pytest.mark.parametrize` 参数化装饰器

```python
@pytest.mark.parametrize('base_info,testcase', get_testcase_yaml("./testcase/Single interface/addUser.yaml"))
```

###### 作用：

- 实现**参数化测试**，即一个测试方法可以运行多组不同的输入数据。
- 每一组`(base_info, testcase)`数据都会触发一次完整的测试执行。

###### 参数说明：

- `'base_info,testcase'`：表示每组测试数据的变量名，在测试函数中作为参数使用。
- `get_testcase_yaml(...)`：调用函数读取指定路径下的YAML文件内容，返回一个包含多个测试用例的列表。

###### 示例（YAML文件内容）：

```yaml
baseInfo:
  api_name: 新增用户
  method: POST
  url: /api/user/add
  header:
    token: ${get_extract_data(token)}

testCase:
  - case_name: 正常场景_新增用户
    data:
      username: testuser001
      password: 123456
    validation:
      - contains: {status_code: 200}
      - contains: {message: 'success'}
    extract:
      userId: $.data.userId
```

##### 测试方法定义

```python
def test_add_user(self, base_info, testcase):
```

- 这是一个**pytest测试方法**，每个参数化的数据都会触发一次该方法的执行。
- `self`表示这是类中的一个实例方法（属于`TestUserManager`类）。
- `base_info`是从YAML文件中加载的接口基本信息。
- `testcase`是从YAML文件中加载的一条测试用例数据。

##### 动态设置Allure报告标题

```python
allure.dynamic.title(testcase['case_name'])
```

###### 作用：

- 在生成的Allure报告中，为当前测试用例设置一个可读性更强的标题。
- 标题内容来自于YAML文件中`testcase['case_name']`字段。

###### 示例效果：

- 如果`case_name`是`"正常场景_新增用户"`，那么在报告中显示的用例名称就是这个值。

##### 执行接口请求和校验

```python
RequestBase().specification_yaml(base_info, testcase)
```

###### 作用：

- 创建`RequestBase`实例，并调用其`specification_yaml()`方法。
- 该方法接收接口基本信息和测试用例数据，并根据其中的配置（如URL、方法、请求头、预期结果等）发送HTTP请求。
- 同时会进行动态参数替换、响应断言、参数提取、日志记录等操作，完成整个接口测试流程。

### 3. 接口处理 `specification_yaml()` 方法

#### 代码示例

```python
def specification_yaml(self, base_info, test_case):
    """
    接口请求处理基本方法
    :param base_info: yaml文件里面的baseInfo
    :param test_case: yaml文件里面的testCase
    :return:
    """
    try:
        params_type = ['data', 'json', 'params']
        url_host = self.conf.get_section_for_data('api_envi', 'host')
        api_name = base_info['api_name']
        allure.attach(api_name, f'接口名称：{api_name}', allure.attachment_type.TEXT)
        url = url_host + base_info['url']
        allure.attach(api_name, f'接口地址：{url}', allure.attachment_type.TEXT)
        method = base_info['method']
        allure.attach(api_name, f'请求方法：{method}', allure.attachment_type.TEXT)
        header = self.replace_load(base_info['header'])
        allure.attach(api_name, f'请求头：{header}', allure.attachment_type.TEXT)
        
        # 处理cookie
        cookie = None
        if base_info.get('cookies') is not None:
            cookie = eval(self.replace_load(base_info['cookies']))
        
        case_name = test_case.pop('case_name')
        allure.attach(api_name, f'测试用例名称：{case_name}', allure.attachment_type.TEXT)
        
        # 处理断言
        val = self.replace_load(test_case.get('validation'))
        test_case['validation'] = val
        validation = eval(test_case.pop('validation'))
        
        # 处理参数提取
        extract = test_case.pop('extract', None)
        extract_list = test_case.pop('extract_list', None)
        
        # 处理接口的请求参数
        for key, value in test_case.items():
            if key in params_type:
                test_case[key] = self.replace_load(value)
        
        # 处理文件上传接口
        file, files = test_case.pop('files', None), None
        if file is not None:
            for fk, fv in file.items():
                allure.attach(json.dumps(file), '导入文件')
                files = {fk: open(fv, mode='rb')}
        
        res = self.run.run_main(name=api_name, url=url, case_name=case_name, header=header, method=method,
                                file=files, cookies=cookie, **test_case)
        status_code = res.status_code
        allure.attach(self.allure_attach_response(res.json()), '接口响应信息', allure.attachment_type.TEXT)
        
        try:
            res_json = json.loads(res.text)  # 把json格式转换成字典
            if extract is not None:
                self.extract_data(extract, res.text)
            if extract_list is not None:
                self.extract_data_list(extract_list, res.text)
            # 处理断言
            self.asserts.assert_result(validation, res_json, status_code)
        except JSONDecodeError as js:
            logs.error('系统异常或接口未请求！')
            raise js
        except Exception as e:
            logs.error(e)
            raise e
    except Exception as e:
        raise e
```

#### `specification_yaml` 方法解析

该方法用于处理YAML文件中定义的接口测试用例，包括请求参数构造、动态变量替换、接口调用、响应断言以及数据提取等核心功能。

##### 方法定义

```python
def specification_yaml(self, base_info, test_case):
    """
    接口请求处理基本方法
    :param base_info: yaml文件里面的baseInfo（接口基本信息）
    :param test_case: yaml文件里面的testCase（测试用例数据）
    :return: 无返回值，但通过断言验证接口行为
    """
```

#### 步骤详解

##### 1. 定义常量与基础配置

```python
params_type = ['data', 'json', 'params']
url_host = self.conf.get_section_for_data('api_envi', 'host')
```

- `params_type`：表示请求参数可能包含的类型字段。
- `url_host`：从配置文件中获取当前环境的主机地址。

##### 2. 提取接口基本信息并添加Allure报告附件

```python
api_name = base_info['api_name']
allure.attach(api_name, f'接口名称：{api_name}', allure.attachment_type.TEXT)
url = url_host + base_info['url']
allure.attach(api_name, f'接口地址：{url}', allure.attachment_type.TEXT)
method = base_info['method']
allure.attach(api_name, f'请求方法：{method}', allure.attachment_type.TEXT)
```

- 从`base_info`中提取接口名称、URL和请求方法。
- 使用`allure.attach`将这些信息附加到Allure报告中，便于测试结果查看。

##### 3. 处理请求头和Cookie

```python
header = self.replace_load(base_info['header'])
allure.attach(api_name, f'请求头：{header}', allure.attachment_type.TEXT)
cookie = None
if base_info.get('cookies') is not None:
    cookie = eval(self.replace_load(base_info['cookies']))
```

- 调用`replace_load`对header进行变量替换（处理`${}`表达式）。
- 如果存在cookies，则同样进行替换并使用`eval`转换为字典格式。

##### 4. 提取测试用例名称并添加报告附件

```python
case_name = test_case.pop('case_name')
allure.attach(api_name, f'测试用例名称：{case_name}', allure.attachment_type.TEXT)
```

- 从`test_case`中提取用例名称并删除原始字段。
- 添加到Allure报告中。

##### 5. 处理断言逻辑

```python
val = self.replace_load(test_case.get('validation'))
test_case['validation'] = val
validation = eval(test_case.pop('validation'))
```

- 替换断言表达式中的变量。
- 使用`eval`执行断言表达式，生成实际断言规则。

##### 6. 提取数据字段（extract / extract_list）

```python
extract = test_case.pop('extract', None)
extract_list = test_case.pop('extract_list', None)
```

- 从`test_case`中提取需要提取的字段名，供后续从响应中提取数据。

##### 7. 处理请求参数（data/json/params）

```python
for key, value in test_case.items():
    if key in params_type:
        test_case[key] = self.replace_load(value)
```

- 遍历所有测试用例参数，若为`data`、`json`或`params`类型，则对其值进行变量替换。

##### 8. 处理文件上传

```python
file, files = test_case.pop('files', None), None
if file is not None:
    for fk, fv in file.items():
        allure.attach(json.dumps(file), '导入文件')
        files = {fk: open(fv, mode='rb')}
```

- 如果存在`files`字段，表示是文件上传接口。
- 使用`open(..., mode='rb')`读取文件内容，并附加到Allure报告中。

##### 9. 发送接口请求

```python
res = self.run.run_main(name=api_name, url=url, case_name=case_name, header=header,
                        method=method, file=files, cookies=cookie, **test_case)
status_code = res.status_code
allure.attach(self.allure_attach_response(res.json()), '接口响应信息', allure.attachment_type.TEXT)
```

- 调用`run_main`方法发送请求。
- 获取状态码和响应内容。
- 将响应内容附加到Allure报告中。

##### 10. 处理响应与断言

```python
try:
    res_json = json.loads(res.text)  # 把json格式转换成字典
    if extract is not None:
        self.extract_data(extract, res.text)
    if extract_list is not None:
        self.extract_data_list(extract_list, res.text)
    # 处理断言
    self.asserts.assert_result(validation, res_json, status_code)
except JSONDecodeError as js:
    logs.error('系统异常或接口未请求！')
    raise js
except Exception as e:
    logs.error(e)
    raise e
```

- 将响应内容转为JSON字典。
- 若有`extract`或`extract_list`，则调用对应方法提取数据。
- 使用断言方法对响应结果进行验证。

##### 11. 异常捕获

```python
except Exception as e:
    raise e
```

- 捕获所有异常并重新抛出，确保测试框架可以正确识别失败情况。

#### 示例说明

假设YAML文件如下：

```yaml
baseInfo:
  api_name: 登录接口
  url: /api/login
  method: POST
  header:
    Content-Type: application/json
    cookies: null

testCase:
  - case_name: 正常登录
    json:
      username: admin
      password: ${get_password()}
    validation:
      - contains: {code: 200}
      - contains: {msg: "success"}
    extract:
      token: $.data.token
```

经过`specification_yaml`处理后：

- `${get_password()}`会被替换成实际密码。
- 请求会携带正确的JSON数据。
- 响应中的`token`会被提取保存到`extract.yaml`。
- 断言会检查是否返回预期结果。

### 4. 动态参数替换 `replace_load()` 方法

#### 代码示例

```python
def replace_load(self, data):
    """
    yaml数据替换解析
    :param data: 需要替换的数据
    :return: 替换后的数据
    """
    str_data = data
    if not isinstance(data, str):
        str_data = json.dumps(data, ensure_ascii=False)
    
    # 循环替换所有 ${} 表达式
    for i in range(str_data.count('${')):
        if '${' in str_data and '}' in str_data:
            # 提取 ${get_extract_data(token)}
            start_index = str_data.index('$')
            end_index = str_data.index('}', start_index)
            ref_all_params = str_data[start_index:end_index + 1]
            
            # 提取函数名：get_extract_data
            func_name = ref_all_params[2:ref_all_params.index("(")]
            
            # 提取参数：token
            func_params = ref_all_params[ref_all_params.index("(") + 1:ref_all_params.index(")")]
            
            # 核心：通过反射机制动态调用DebugTalk类的方法
            extract_data = getattr(DebugTalk(), func_name)(*func_params.split(',') if func_params else "")
            
            # 替换原始表达式
            str_data = str_data.replace(ref_all_params, str(extract_data))
    
    # 还原数据类型
    if data and isinstance(data, dict):
        data = json.loads(str_data)
    else:
        data = str_data
    return data
```

#### `replace_load` 方法解析

该方法是框架的核心机制之一，用于实现动态参数替换，支持接口关联、时间戳生成、加密等功能。

##### 方法定义

```python
def replace_load(self, data):
    """
    yaml数据替换解析
    :param data: 需要替换的数据（可以是字符串、字典等）
    :return: 替换后的数据
    """
```

##### 步骤详解

###### 1. 数据类型转换

```python
str_data = data
if not isinstance(data, str):
    str_data = json.dumps(data, ensure_ascii=False)
```

- 将输入数据转换为字符串格式，便于处理。

###### 2. 循环替换所有`${}`表达式

```python
for i in range(str_data.count('${')):
    if '${' in str_data and '}' in str_data:
        start_index = str_data.index('$')
        end_index = str_data.index('}', start_index)
        ref_all_params = str_data[start_index:end_index + 1]
```

- 检测字符串中的`${...}`模式。
- 提取完整的表达式，如`${get_extract_data(token)}`。

###### 3. 提取函数名和参数

```python
func_name = ref_all_params[2:ref_all_params.index("(")]
func_params = ref_all_params[ref_all_params.index("(") + 1:ref_all_params.index(")")]
```

- 从表达式中提取函数名（如`get_extract_data`）。
- 提取函数参数（如`token`）。

###### 4. 反射调用DebugTalk类的方法

```python
extract_data = getattr(DebugTalk(), func_name)(*func_params.split(',') if func_params else "")
```

- 使用`getattr()`反射机制动态调用`DebugTalk`类的方法。
- 支持多参数调用（通过逗号分隔）。

###### 5. 替换原始表达式

```python
str_data = str_data.replace(ref_all_params, str(extract_data))
```

- 用函数返回值替换原始的`${}`表达式。

###### 6. 还原数据类型

```python
if data and isinstance(data, dict):
    data = json.loads(str_data)
else:
    data = str_data
return data
```

- 如果原始数据是字典类型，则还原为字典格式。

##### 示例说明

假设YAML中有如下配置：

```yaml
header:
  token: ${get_extract_data(token)}
data:
  timestamp: ${timestamp()}
  phone: ${random_phone()}
```

经过`replace_load`处理后：

- `${get_extract_data(token)}`会被替换为`extract.yaml`中存储的token值。
- `${timestamp()}`会被替换为当前时间戳。
- `${random_phone()}`会被替换为随机生成的手机号。

## 四、总结

本框架采用**数据驱动测试**的设计理念，将测试数据（YAML）与测试逻辑（Python代码）完全分离，具有以下特点：

1. **高度封装**：核心功能封装在`RequestBase`类中，测试人员只需编写YAML文件即可完成接口测试。
2. **动态参数机制**：通过`${}`语法实现参数的动态替换，支持接口关联、时间戳生成、加密等操作。
3. **接口自动关联**：自动提取前一个接口的返回值（如token、userId），存储到`extract.yaml`供后续接口使用。
4. **多种断言方式**：支持包含断言、相等断言、不相等断言、响应值断言、数据库断言等5种策略。
5. **完整报告**：集成Allure生成可视化报告，展示每个接口的请求参数、响应内容、断言结果。

通过本框架，测试人员不需要掌握Python编程，只需会编写YAML文件就能完成接口自动化测试，大幅降低了测试门槛，提升了测试效率。
