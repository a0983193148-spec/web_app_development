# 流程圖設計 — 食譜搜尋系統

## 1. 使用者流程圖（User Flow）

以下流程圖描述使用者從進入網站開始，可以進行的所有主要操作路徑：

```mermaid
flowchart LR
    A(["🧑‍🍳 使用者開啟網頁"]) --> B["首頁<br/>搜尋框 + 分類瀏覽"]

    B --> C{"要執行什麼操作？"}

    C -->|"搜尋食譜"| D["輸入關鍵字"]
    D --> E["顯示搜尋結果列表"]
    E --> F["點擊食譜"]
    F --> G["食譜詳情頁"]

    C -->|"瀏覽分類"| H["選擇分類類別"]
    H --> I["顯示該分類食譜列表"]
    I --> F

    C -->|"上傳食譜"| J["填寫食譜表單<br/>名稱/食材/步驟/照片"]
    J --> K{"表單是否填寫完整？"}
    K -->|"否"| L["顯示錯誤提示"]
    L --> J
    K -->|"是"| M["儲存食譜到資料庫"]
    M --> N["顯示成功訊息"]
    N --> G

    C -->|"食材推薦"| O["輸入手邊食材"]
    O --> P["顯示推薦食譜列表"]
    P --> F

    C -->|"我的收藏"| Q["顯示收藏食譜列表"]
    Q --> F

    G --> R{"要進行什麼操作？"}
    R -->|"收藏"| S["加入收藏清單"]
    S --> G
    R -->|"取消收藏"| T["從收藏清單移除"]
    T --> G
    R -->|"編輯"| U["編輯食譜表單"]
    U --> V["儲存修改"]
    V --> G
    R -->|"刪除"| W["確認刪除？"]
    W -->|"取消"| G
    W -->|"確認"| X["刪除食譜"]
    X --> B
    R -->|"返回"| B
```

### 流程說明

1. **進入首頁**：使用者開啟網頁後看到搜尋框與分類瀏覽區
2. **搜尋食譜**：輸入關鍵字 → 查看搜尋結果 → 點擊進入詳情頁
3. **瀏覽分類**：選擇料理分類 → 查看該分類食譜 → 點擊進入詳情頁
4. **上傳食譜**：填寫表單 → 驗證 → 儲存到資料庫 → 跳轉到詳情頁
5. **食材推薦**：輸入手邊食材 → 系統比對推薦 → 查看結果
6. **我的收藏**：瀏覽已收藏食譜 → 點擊進入詳情頁
7. **食譜操作**：在詳情頁可進行收藏 / 編輯 / 刪除等操作

---

## 2. 系統序列圖（Sequence Diagram）

### 2.1 搜尋食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Recipe Model
    participant DB as 🗄️ SQLite

    User->>Browser: 在搜尋框輸入「番茄」並按下搜尋
    Browser->>Flask: GET /recipes/search?q=番茄
    Flask->>Model: Recipe.search("番茄")
    Model->>DB: SELECT * FROM recipes WHERE name LIKE '%番茄%'
    DB-->>Model: 回傳匹配的食譜資料
    Model-->>Flask: 回傳食譜列表
    Flask-->>Browser: render_template("recipe/list.html", recipes=結果)
    Browser-->>User: 顯示搜尋結果頁面
```

### 2.2 上傳食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Recipe Model
    participant DB as 🗄️ SQLite
    participant FS as 📁 檔案系統

    User->>Browser: 點擊「上傳食譜」
    Browser->>Flask: GET /recipes/create
    Flask-->>Browser: render_template("recipe/create.html")
    Browser-->>User: 顯示上傳表單

    User->>Browser: 填寫食譜資訊並上傳照片，按下送出
    Browser->>Flask: POST /recipes/create (表單資料 + 圖片)
    Flask->>Flask: 驗證表單資料與圖片格式
    alt 驗證失敗
        Flask-->>Browser: 顯示錯誤訊息，回到表單頁
        Browser-->>User: 看到錯誤提示
    else 驗證成功
        Flask->>FS: 儲存圖片到 uploads/ 資料夾
        FS-->>Flask: 回傳圖片檔案路徑
        Flask->>Model: Recipe.create(名稱, 食材, 步驟, 圖片路徑...)
        Model->>DB: INSERT INTO recipes (...) VALUES (...)
        DB-->>Model: 回傳新食譜 ID
        Model-->>Flask: 回傳新食譜資料
        Flask-->>Browser: redirect("/recipes/<id>")
        Browser-->>User: 顯示新建立的食譜詳情頁
    end
```

### 2.3 收藏食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Favorite Model
    participant DB as 🗄️ SQLite

    User->>Browser: 在食譜詳情頁點擊「收藏」按鈕
    Browser->>Flask: POST /favorites/add (recipe_id)
    Flask->>Model: Favorite.add(recipe_id)
    Model->>DB: INSERT INTO favorites (recipe_id) VALUES (...)
    DB-->>Model: 儲存成功
    Model-->>Flask: 回傳成功
    Flask-->>Browser: redirect 回食譜詳情頁
    Browser-->>User: 顯示「已收藏」狀態
```

### 2.4 編輯食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Recipe Model
    participant DB as 🗄️ SQLite

    User->>Browser: 在食譜詳情頁點擊「編輯」
    Browser->>Flask: GET /recipes/<id>/edit
    Flask->>Model: Recipe.get_by_id(id)
    Model->>DB: SELECT * FROM recipes WHERE id = ?
    DB-->>Model: 回傳食譜資料
    Model-->>Flask: 回傳食譜物件
    Flask-->>Browser: render_template("recipe/edit.html", recipe=資料)
    Browser-->>User: 顯示預填好的編輯表單

    User->>Browser: 修改內容後按下儲存
    Browser->>Flask: POST /recipes/<id>/edit (更新的表單資料)
    Flask->>Model: Recipe.update(id, 更新資料)
    Model->>DB: UPDATE recipes SET ... WHERE id = ?
    DB-->>Model: 更新成功
    Model-->>Flask: 回傳成功
    Flask-->>Browser: redirect("/recipes/<id>")
    Browser-->>User: 顯示更新後的食譜詳情頁
```

### 2.5 刪除食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Recipe Model
    participant DB as 🗄️ SQLite

    User->>Browser: 在食譜詳情頁點擊「刪除」
    Browser->>User: 彈出確認對話框：「確定要刪除嗎？」
    alt 取消刪除
        User->>Browser: 按下「取消」
        Browser-->>User: 留在食譜詳情頁
    else 確認刪除
        User->>Browser: 按下「確認」
        Browser->>Flask: POST /recipes/<id>/delete
        Flask->>Model: Recipe.delete(id)
        Model->>DB: DELETE FROM recipes WHERE id = ?
        DB-->>Model: 刪除成功
        Model-->>Flask: 回傳成功
        Flask-->>Browser: redirect("/")
        Browser-->>User: 回到首頁
    end
```

### 2.6 食材推薦食譜流程

```mermaid
sequenceDiagram
    actor User as 🧑‍🍳 使用者
    participant Browser as 🌐 瀏覽器
    participant Flask as 🎮 Flask Route
    participant Model as 📦 Recipe Model
    participant DB as 🗄️ SQLite

    User->>Browser: 輸入手邊食材「雞蛋, 番茄, 洋蔥」
    Browser->>Flask: GET /recipes/recommend?ingredients=雞蛋,番茄,洋蔥
    Flask->>Flask: 解析食材清單
    Flask->>Model: Recipe.find_by_ingredients(["雞蛋", "番茄", "洋蔥"])
    Model->>DB: SELECT recipes 透過 JOIN ingredients 比對
    DB-->>Model: 回傳匹配的食譜（依匹配度排序）
    Model-->>Flask: 回傳推薦食譜列表
    Flask-->>Browser: render_template("recipe/list.html", recipes=推薦結果)
    Browser-->>User: 顯示推薦食譜列表
```

---

## 3. 功能清單對照表

| 功能 | URL 路徑 | HTTP 方法 | 說明 |
|------|----------|-----------|------|
| 首頁 | `/` | GET | 顯示搜尋框、分類瀏覽、推薦食譜 |
| 搜尋食譜 | `/recipes/search` | GET | 依關鍵字搜尋食譜，回傳結果列表 |
| 食譜列表 | `/recipes` | GET | 顯示所有食譜列表 |
| 食譜詳情 | `/recipes/<id>` | GET | 顯示單一食譜的完整資訊 |
| 新增食譜（表單） | `/recipes/create` | GET | 顯示上傳食譜的空白表單 |
| 新增食譜（送出） | `/recipes/create` | POST | 接收表單資料，建立新食譜 |
| 編輯食譜（表單） | `/recipes/<id>/edit` | GET | 顯示預填資料的編輯表單 |
| 編輯食譜（送出） | `/recipes/<id>/edit` | POST | 接收更新資料，修改食譜 |
| 刪除食譜 | `/recipes/<id>/delete` | POST | 刪除指定食譜 |
| 分類瀏覽 | `/categories/<name>` | GET | 顯示指定分類下的食譜列表 |
| 食材推薦 | `/recipes/recommend` | GET | 根據輸入的食材推薦食譜 |
| 加入收藏 | `/favorites/add` | POST | 將食譜加入收藏清單 |
| 取消收藏 | `/favorites/remove` | POST | 將食譜從收藏清單移除 |
| 我的收藏 | `/favorites` | GET | 顯示所有已收藏的食譜 |
