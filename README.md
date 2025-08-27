# Concert Ticket Booking Service
콘서트 티켓 예매 서비스입니다. 사용자는 좌석 예약 시 미리 충전한 잔액을 이용하며 결제가 이루어지지 않더라도 일정 시간동안 다른 유저가 해당 좌석에 접근할 수 없습니다.

# API Specification
<details>
 <summary><code>POST</code> <code><b>/queue/token</b></code> <code>대기열 토큰 발급 API</code></summary>

#### Description
> 사용자 대기열 토큰을 발급받습니다.

#### Request
> ```json
> {
>   "userId": "string (UUID)"
> }
> ```

#### Response
> **Success (200)**
> ```json
> {
>   "token": "string (JWT)",
>   "queuePosition": "number",
>   "estimatedWaitTime": "number (minutes)",
>   "status": "WAITING | ACTIVE"
> }
> ```
> 
> **Error (400)**
> ```json
> {
>   "statusCode": 400,
>   "message": "Invalid user ID format",
>   "error": "Bad Request"
> }
> ```
</details>

<details>
 <summary><code>GET</code> <code><b>/queue/status</b></code> <code>대기열 상태 조회 API</code></summary>

#### Description
> 현재 사용자의 대기열 상태를 조회합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`

#### Response
> **Success (200)**
> ```json
> {
>   "queuePosition": "number",
>   "estimatedWaitTime": "number (minutes)",
>   "status": "WAITING | ACTIVE | EXPIRED"
> }
> ```
> 
> **Error (401)**
> ```json
> {
>   "statusCode": 401,
>   "message": "Invalid or expired token",
>   "error": "Unauthorized"
> }
> ```
</details>

<details>
 <summary><code>GET</code> <code><b>/concerts/{concertId}/available-dates</b></code> <code>예약 가능 날짜 조회 API</code></summary>

#### Description
> 콘서트의 예약 가능한 날짜 목록을 조회합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Path Parameters
> - `concertId` (number): 콘서트 ID

#### Response
> **Success (200)**
> ```json
> {
>   "concertId": "number",
>   "availableDates": "string[] (YYYY-MM-DD)"
> }
> ```
</details>

<details>
 <summary><code>GET</code> <code><b>/concerts/{concertId}/available-seats</b></code> <code>좌석 조회 API</code></summary>

#### Description
> 특정 날짜의 예약 가능한 좌석 정보를 조회합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Path Parameters
> - `concertId` (number): 콘서트 ID
> 
> ##### Query Parameters
> - `date` (string, required): 조회할 날짜 (YYYY-MM-DD)

#### Response
> **Success (200)**
> ```json
> {
>   "concertId": "number",
>   "date": "string (YYYY-MM-DD)",
>   "seats": [
>     {
>       "seatNumber": "number",
>       "price": "number"
>     }
>   ]
> }
> ```
> 
> **Error (404)**
> ```json
> {
>   "statusCode": 404,
>   "message": "Concert not found",
>   "error": "Not Found"
> }
> ```
</details>

<details>
 <summary><code>POST</code> <code><b>/reservations</b></code> <code>좌석 예약 요청 API</code></summary>

#### Description
> 좌석을 임시 예약합니다. (5분간 임시 배정)

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Body
> ```json
> {
>   "concertId": "number",
>   "date": "string (YYYY-MM-DD)",
>   "seatNumber": "number"
> }
> ```

#### Response
> **Success (201)**
> ```json
> {
>   "reservationId": "string (UUID)",
>   "concertId": "number",
>   "date": "string (YYYY-MM-DD)",
>   "seatNumber": "number",
>   "status": "TEMPORARILY_RESERVED",
>   "expiresAt": "string (ISO 8601)",
>   "price": "number"
> }
> ```
> 
> **Error (409)**
> ```json
> {
>   "statusCode": 409,
>   "message": "Seat is already reserved",
>   "error": "Conflict"
> }
> ```
> 
> **Error (403)**
> ```json
> {
>   "statusCode": 403,
>   "message": "Queue token is not active",
>   "error": "Forbidden"
> }
> ```
</details>

<details>
 <summary><code>GET</code> <code><b>/users/{userId}/balance</b></code> <code>잔액 조회 API</code></summary>

#### Description
> 사용자의 현재 잔액을 조회합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Path Parameters
> - `userId` (string): 사용자 UUID

#### Response
> **Success (200)**
> ```json
> {
>   "userId": "string (UUID)",
>   "balance": "number"
> }
> ```
</details>

<details>
 <summary><code>POST</code> <code><b>/users/{userId}/charge</b></code> <code>잔액 충전 API</code></summary>

#### Description
> 사용자 계정에 잔액을 충전합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Path Parameters
> - `userId` (string): 사용자 UUID
> 
> ##### Body
> ```json
> {
>   "amount": "number (positive)"
> }
> ```

#### Response
> **Success (200)**
> ```json
> {
>   "userId": "string (UUID)",
>   "chargedAmount": "number",
>   "balance": "number",
>   "chargedAt": "string (ISO 8601)"
> }
> ```
> 
> **Error (400)**
> ```json
> {
>   "statusCode": 400,
>   "message": "Amount must be positive",
>   "error": "Bad Request"
> }
> ```
</details>

<details>
 <summary><code>POST</code> <code><b>/payments</b></code> <code>결제 API</code></summary>

#### Description
> 예약된 좌석에 대한 결제를 처리합니다.

#### Request
> ##### Headers
> - `Authorization: Bearer {queue_token}`
> 
> ##### Body
> ```json
> {
>   "reservationId": "string (UUID)"
> }
> ```

#### Response
> **Success (200)**
> ```json
> {
>   "paymentId": "string (UUID)",
>   "reservationId": "string (UUID)",
>   "userId": "string (UUID)",
>   "amount": "number",
>   "status": "COMPLETED",
>   "paidAt": "string (ISO 8601)"
> }
> ```
> 
> **Error (400)**
> ```json
> {
>   "statusCode": 400,
>   "message": "Reservation has expired",
>   "error": "Bad Request"
> }
> ```
> 
> **Error (402)**
> ```json
> {
>   "statusCode": 402,
>   "message": "Insufficient balance",
>   "error": "Payment Required"
> }
> ```
</details>

# ERD
# Infrastructure