# Python 函式參數與引數完整說明手冊 (`function.ipynb`)

> 本說明書針對 [function.ipynb](file:///C:/Users/User/Desktop/暫存資料夾/test__1111/2026_11_18/function.ipynb) 筆記本內容進行詳細剖析，完整梳理 Python 函式在「定義參數 (Parameters)」與「傳遞引數 (Arguments)」時的核心規則與進階機制。

---

## 目錄
1. [Notebook 核心主題概述](#1-notebook-核心主題概述)
2. [基礎函式與引數傳遞](#2-基礎函式與引數傳遞)
   - [2.1 基礎定義與呼叫 (`greet`)](#21-基礎定義與呼叫-greet)
   - [2.2 位置引數 vs 引數名稱 (`describe_pet`)](#22-位置引數-vs-引數名稱-describe_pet)
   - [2.3 混用規則：位置引數必須在最前](#23-混用規則位置引數必須在最前)
3. [預設值引數應用 (`order_drink`)](#3-預設值引數應用-order_drink)
4. [特殊語法限制：`/` 與 `*`](#4-特殊語法限制--與-)
   - [4.1 僅限位置引數：`/` (`calc_bmi`)](#41-僅限位置引數--calc_bmi)
   - [4.2 僅限關鍵字引數：`*` (`send_email`)](#42-僅限關鍵字引數--send_email)
   - [4.3 綜合三段式定義 (`register`)](#43-綜合三段式定義-register)
5. [不定量引數打包：`*args` 與 `**kwargs`](#5-不定量引數打包-args-與-kwargs)
   - [5.1 `*args`：打包多餘位置引數為 Tuple (`sum_scores`)](#51-args打包多餘位置引數為-tuple-sum_scores)
   - [5.2 `**kwargs`：打包多餘關鍵字引數為 Dict (`build_profile`)](#52-kwargs打包多餘關鍵字引數為-dict-build_profile)
6. [標準函式庫的實務驗證 (`print` 與 `sorted`)](#6-標準函式庫的實務驗證-print-與-sorted)
7. [速查對照表與常見錯誤彙整](#7-速查對照表與常見錯誤彙整)

---

## 1. Notebook 核心主題概述

在 Python 中，「如何把資料傳入函式」是寫出靈活、強韌代碼的關鍵基礎：
- **參數 (Parameter)**：定義函式時寫在括號內的變數名稱。
- **引數 (Argument)**：呼叫函式時實際傳入的值或資料。

本筆記本循序漸進展示了從「最基礎的位置傳遞」到「現代 Python 3.8+ 的 `/` 與 `*` 強制呼叫規範」，以及「動態打包 `*args` / `**kwargs`」的高階用法。

---

## 2. 基礎函式與引數傳遞

### 2.1 基礎定義與呼叫 (`greet`)
最基礎的單一參數函式，透過傳入字串產生問候訊息：

```python
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# 輸出: Hello, Alice!
```

---

### 2.2 位置引數 vs 引數名稱 (`describe_pet`)
當函式有多個參數時，可透過以下兩種方式傳遞：

```python
def describe_pet(animal, name):
    print(f"我養了一隻{animal} ,牠叫做{name}。")
```

#### ① 引數位置 (Positional Arguments)
依照定義時的順序對應傳入。**順序一旦顛倒，邏輯就會錯亂**：
```python
describe_pet("狗", "小白")  # 輸出: 我養了一隻狗 ,牠叫做小白。
describe_pet("小白", "狗")  # 輸出: 我養了一隻小白 ,牠叫做狗。 (順序顛倒)
```

#### ② 引數名稱 / 關鍵字引數 (Keyword Arguments)
明確指定 `參數名=值`。此時**順序不受限制**：
```python
describe_pet(animal="狗", name="小白")  # 輸出: 我養了一隻狗 ,牠叫做小白。
describe_pet(name="小白", animal="狗")  # 輸出: 我養了一隻狗 ,牠叫做小白。
```

---

### 2.3 混用規則：位置引數必須在最前
呼叫函式時若同時混用「位置引數」與「關鍵字引數」，**所有位置引數必須寫在關鍵字引數的前面**。

```python
# ✅ 正確：先傳位置引數，再傳關鍵字引數
describe_pet("狗", name="小白")

# ❌ 錯誤：位置引數不可放在關鍵字引數後面 (會觸發 SyntaxError)
# describe_pet(animal="狗", "小白")
```

---

## 3. 預設值引數應用 (`order_drink`)

當參數有設定「預設值（Default Argument）」時，呼叫者可以只覆寫想變更的項目，簡化呼叫流程：

```python
def order_drink(drink, ice="正常冰", sugar="正常糖"):
    print(f"點餐: {drink} ({ice}/{sugar})")

order_drink("珍珠奶茶")                          # 全使用預設值
order_drink("珍珠奶茶", "去冰")                   # 依照位置覆寫 ice
order_drink("珍珠奶茶", sugar="半糖")             # 依照名稱覆寫 sugar，ice 保持預設
order_drink("綠茶", sugar="半糖", ice="去冰")      # 搭配關鍵字引數，順序可互換
```

**輸出結果**：
```text
點餐: 珍珠奶茶 (正常冰/正常糖)
點餐: 珍珠奶茶 (去冰/正常糖)
點餐: 珍珠奶茶 (正常冰/半糖)
點餐: 綠茶 (去冰/半糖)
```

---

## 4. 特殊語法限制：`/` 與 `*`

為了提高 API 設計的嚴謹性與清晰度，Python 提供了 `/` 與 `*` 作為分隔符號，強制約束引數的呼叫風格：

```text
def func(僅限位置參數, /, 位置或名稱皆可, *, 僅限關鍵字參數):
```

```mermaid
flowchart LR
    A["/ 符號左側\n(Positional-Only)\n只能用位置傳入\n禁止寫名稱"] --> B["中間區域\n(Positional or Keyword)\n位置或名稱皆可"]
    B --> C["* 符號右側\n(Keyword-Only)\n必須用名稱傳入\n禁止純靠位置"]
```

---

### 4.1 僅限位置引數：`/` (`calc_bmi`)
寫在 `/` **左邊**的所有參數，只能透過位置傳入，呼叫時**禁止**寫出參數名稱：

```python
def calc_bmi(weight, height, /):
    bmi = weight / (height ** 2)
    print(f"BMI 為：{bmi:.1f}")

calc_bmi(65, 1.75)   # ✅ 正確，輸出: BMI 為：21.2
# calc_bmi(weight=65, height=1.75) # ❌ TypeError: calc_bmi() got some positional-only arguments passed as keyword arguments
```
* **好處**：未來若函式內部想要修改變數名稱（例如 `weight` 改為 `w`），完全不影響呼叫端。

---

### 4.2 僅限關鍵字引數：`*` (`send_email`)
寫在獨立星號 `*` **右邊**的所有參數，呼叫時**必須明確寫出引數名稱**：

```python
def send_email(to, *, subject, urgent=False):
    print(f"寄給：{to}｜主旨：{subject}｜急件：{urgent}")

send_email("student@example.com", subject="課堂通知", urgent=True)
# 輸出: 寄給：student@example.com｜主旨：課堂通知｜急件：True
```
* **好處**：避免重要或容易混淆的設定值（如布林值開關）在傳入時失去語意。

---

### 4.3 綜合三段式定義 (`register`)
結合 `/` 與 `*`，可精準劃分三種傳遞區間：

```python
def register(id_no, /, name, *, role="student"):
    print(f"編號：{id_no}｜姓名：{name}｜身分：{role}")

# id_no 只能位置傳入, name 可位置也可名稱, role 必須名稱傳入
register("A01", "小明", role="leader")
# 輸出: 編號：A01｜姓名：小明｜身分：leader
```

---

## 5. 不定量引數打包：`*args` 與 `**kwargs`

當設計函式時無法預先得知會傳入多少引數時，使用單星號 `*` 或雙星號 `**` 將額外的引數自動「打包 (Packing)」。

### 5.1 `*args`：打包多餘位置引數為 Tuple (`sum_scores`)
* 接收任意數量的「位置引數」，並自動包裝成一個 `tuple`：

```python
def sum_scores(*scores):
    print("收到的資料：", scores, type(scores))
    return sum(scores)

print(sum_scores(80, 90, 85))  # 收到的資料： (80, 90, 85) <class 'tuple'> -> 回傳 255
print(sum_scores(80, 90))      # 收到的資料： (80, 90) <class 'tuple'>     -> 回傳 170
print(sum_scores())            # 收到的資料： () <class 'tuple'>           -> 回傳 0
```

---

### 5.2 `**kwargs`：打包多餘關鍵字引數為 Dict (`build_profile`)
* 接收任意數量的「關鍵字引數」，並自動包裝成一個 `dict`：

```python
def build_profile(name, **info):
    print("姓名：", name)
    print("其他資料：", info, type(info))

build_profile("小華", city="台北", course="Python")
# 輸出:
# 姓名： 小華
# 其他資料： {'city': '台北', 'course': 'Python'} <class 'dict'>
```

---

## 6. 標準函式庫的實務驗證 (`print` 與 `sorted`)

本單元展示 Python 內建函式正是上述規則的最佳範例：

### ① `print(*args, sep=' ', end='\n')`
* 前方的多個印出項目就是利用 `*args` 接收任意多個引數。
* 後方的 `sep` 與 `end` 為關鍵字引數，用於控制間隔與結尾：

```python
print('a', 'b', 1, 67, 89)                   # 預設空白隔開: a b 1 67 89
print('a', 'b', 1, 67, 89, sep='-')          # 自訂分隔符: a-b-1-67-89
print('a', 'b', 1, 67, 89, end='讚啦!\n')    # 自訂結尾字元: a b 1 67 89讚啦!
```

### ② `sorted(iterable, *, key=None, reverse=False)`
* 排序對象為位置引數。
* 排序規則（`key` 與 `reverse`）強制放在 `*` 後方，呼叫時必須使用關鍵字引數指名：

```python
sorted([8, 92, 63, 75], reverse=True)
# 輸出: [92, 75, 63, 8]
```

---

## 7. 速查對照表與常見錯誤彙整

### 語法機制速查表

| 語法型態 | 範例定義語法 | 呼叫時限制 | 打包型態 |
| :--- | :--- | :--- | :--- |
| **位置引數** | `def f(a, b):` | 按順序傳入（順序不可錯） | 無 |
| **關鍵字引數** | `f(a=1, b=2)` | 須指定參數名稱（順序可調） | 無 |
| **預設值** | `def f(a=1):` | 可傳可不傳 | 無 |
| **僅限位置 (`/`)** | `def f(a, /):` | **嚴格禁止**寫參數名稱 `f(a=1)` ❌ | 無 |
| **僅限關鍵字 (`*`)**| `def f(*, key):` | **嚴格禁止**純位置傳入 `f(1)` ❌ | 無 |
| **位置打包 (`*args`)** | `def f(*args):` | 可傳入 0 到多個位置引數 | `tuple` |
| **關鍵字打包 (`**kwargs`)** | `def f(**kwargs):` | 可傳入 0 到多個名稱引數 | `dict` |

### 常見錯誤提示 (Common Errors)
1. **`SyntaxError: positional argument follows keyword argument`**
   - 原因：呼叫時把「位置引數」寫在「關鍵字引數」後面（如：`f(x=1, 2)`）。
2. **`TypeError: ... got some positional-only arguments passed as keyword arguments`**
   - 原因：對 `/` 左側的參數傳遞了參數名稱（如：`calc_bmi(weight=60, height=1.7)`）。
3. **`TypeError: ... takes 1 positional argument but 2 were given`**
   - 原因：對 `*` 右側的參數只用了位置傳遞，未指定名稱。
