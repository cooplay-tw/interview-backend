### **軟體工程實習生（後端）面試測試題**  

#### **測試目標**
本測試旨在考察候選人的後端開發能力，包括**基礎程式設計、API 設計、即時通訊、身份驗證、系統部署**等關鍵技能。測試將分為三個部分，並包含一個進階挑戰（Bonus）。候選人可根據自身專長選擇不同題目進行實作，並且可自由選擇開發語言與框架（Node.js、Python、Go、Java 等皆可），可以使用ChatGPT、DeepSeek、Claude等工具進行開發，不限制使用的工具。

---

## **題目 1：即時通訊後端（基礎）**
### **目標**
開發一個簡單的即時通訊後端，允許用戶透過 WebSocket 互相傳送訊息。

### **需求**
1. **WebSocket 連線機制**
   - 用戶透過 WebSocket 連線到server時，必須在 URL 指定自己的身份：
     ```
     ws://localhost:8080/ws?id=userA
     ```
   - Server 應識別該用戶，並將其連線保持在server中。

2. **點對點訊息傳遞**
   - 當用戶發送訊息時，應該指定接收者，並使用 JSON 格式：
     ```json
     { "to": "userB", "message": "Hello! userB" }
     ```
   - Server 需解析該訊息，並將其轉發給 `to` 指定的用戶。
   - 接收者(如 `userB`)可能會有多個連線，Server 應將訊息傳送給所有連線。

3. **錯誤處理**
   - 若接收者（如 `userB`）尚未連線，應在 Server log 中記錄錯誤：
     ```
     [ERROR] userB 尚未連線，無法傳送訊息
     ```
   - 若有其他錯誤，則視情況適當在 Server log 中記錄錯誤

### **測試方式**
- 測試目標用戶已連線時，Server 是否正確轉發訊息。
- 測試目標用戶userB有2個連線時，Server 是否能正確將訊息傳送給所有連線。
- 測試目標用戶未連線時，Server 是否正確記錄錯誤。

---

## **題目 2：後端系統 Docker 化（進階）**
### **目標**
將開發好的 WebSocket Server 透過 Docker 部署，使其能夠在容器環境中運行。

### **需求**
1. **撰寫 Dockerfile**
   - 使用適當的基礎映像檔（Node.js、Python、Go 等）。
   - 確保所有運行所需的依賴皆可在容器內正確安裝。

2. **容器啟動**
   - 確保容器啟動後，WebSocket Server 能夠正常提供服務，並允許客戶端連線：
     ```
     docker build -t websocket-server .
     docker run -p 8080:8080 websocket-server
     ```

### **測試方式**
- 本機使用 `docker build` 建置映像檔，並用 `docker run` 啟動 Server。
- 測試 WebSocket 連線，確保功能與題目 1 一致。

---

## **題目 3：身份驗證（Bonus）**
### **目標**
在 WebSocket Server 加入身份驗證機制，確保只有通過驗證的用戶才能進行訊息傳遞。

### **需求**
1. **註冊 API**
   - 新增一個 HTTP API `/api/register` 來讓用戶註冊，並返回唯一 Token：
     ```json
     { "id": "userA", "token": "<RANDOM_GENERATED_TOKEN>" }
     ```

2. **WebSocket 連線時驗證**
   - 用戶在 WebSocket 連線時，必須附帶 Token：
     ```
     ws://localhost:8080/ws?id=userA&token=xxx
     ```
   - Server 需驗證 Token 是否有效，並確認其對應的用戶 ID。

3. **訊息傳遞權限**
   - 只有通過身份驗證的用戶，才能透過 WebSocket 傳送訊息。
   - 若 Token 無效，應拒絕 WebSocket 連線或直接關閉連線。

4. **錯誤處理**
   - 若 Token 不存在或錯誤，應返回適當錯誤訊息。
   - 不需實作 Token 過期機制，只需確保 Token 正確時能夠通過驗證。

### **測試方式**
- 透過 `/api/register` 註冊用戶，獲取 Token。
- 測試攜帶正確 Token 連線，應允許進行訊息傳遞。
- 測試 Token 無效或缺失時，應拒絕 WebSocket 連線。

---

## **題目4. 多 Server 互通 (Bonus)**
### **目標**
讓多個 WebSocket Server 之間能夠互相轉發訊息，確保分布式環境下的通訊正常。

### **需求**
1. **架構設計**
   - 假設有兩個獨立的 WebSocket Server（Server A & Server B），用戶可能連線到不同的 Server：
     ```
     Server A → userA
     Server B → userB
     ```
   - 當 userA 發送訊息給 userB，Server A 應該能夠透過**跨伺服器通訊機制**（如 Redis Pub/Sub、RabbitMQ 或 HTTP API）將訊息傳遞給 Server B，最終送達 userB。
   即：`userA -> Server A -> Server B -> userB`

2. **跨伺服器通訊**
   - 使用**Redis Pub/Sub** 或 **RabbitMQ** 來實作 Server 之間的訊息轉發機制。

3. **錯誤處理**
   - 若跨伺服器訊息傳遞失敗，應適當記錄錯誤。

### **測試方式**
- 部署兩個 WebSocket Server，確保用戶分別連線到不同 Server。
- 測試跨 Server 之間的訊息傳遞是否能正確進行。

---

## **評分標準**
| **項目**              | **評估重點** |
|---------------------|------------|
| **程式碼品質**       | 變數命名清楚、架構設計合理、可讀性高 |
| **錯誤處理**       | 能適當處理錯誤狀況，避免程式崩潰 |
| **測試與 Debug 能力** | 能有效測試與排查問題 |
| **系統設計能力**     | 具備良好的架構思維與擴展性考量 |
| **學習與適應能力**   | 能快速理解問題，應對沒有接觸過的新技術 |

---
