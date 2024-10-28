# Distributed Credit Card Services Using Microservices Architecture
### **Objective:**
Design and implement a microservices-based system to manage credit card issuance, billing, payments, and user registration for customers. The project will demonstrate key concepts such as microservices architecture, service discovery, and transaction management.

### **Core Components:**
1. **User Registration Service**
2. **Credit Card Service**
3. **Payments Service**
4. **Spring Cloud Gateway (API Gateway)**
5. **Eureka Service Discovery**

---

### **Requirements:**

## **1. User Registration Service:**
- **Functionalities:**
    - Customers can register to use the system.
    - Two roles exist: `ADMIN` and `CUSTOMER`, with initial data setup creating the `ADMIN` role.
- **User Details Required for Registration:**
    - Name
    - Email (unique, validated as per standard email format)
    - Password (stored securely; must meet complexity requirements such as at least 8 characters, one uppercase letter, one number, and one special character)
    - Aadhaar Number (12-digit unique identifier; validated to be a 12-digit number)
    - PAN Card Number (10-digit unique alphanumeric identifier; validated as per standard PAN format: 5 letters, 4 numbers, 1 letter, e.g., ABCDE1234F)
    - Address (non-empty)
    - Mobile Number (10-digit number, validated with a regex for Indian mobile number formats)
    - Job Role (e.g., Salaried, Freelancer, Student)
    - Monthly Income (validated to be a positive integer)
- **Endpoints:**
    - **`/register` (POST):**
        - Allows new users to register as `CUSTOMER`.
        - **Request Body:**
          ```json
          {
            "name": "John Doe",
            "email": "john.doe@example.com",
            "password": "Secure@123",
            "aadhaarNumber": "123456789012",
            "panCardNumber": "ABCDE1234F",
            "address": "123 Main St, City, Country",
            "mobileNumber": "9876543210",
            "jobRole": "Salaried",
            "monthlyIncome": 5000
          }
          ```
        - **Data Validation:** All fields are validated. If the data is invalid (e.g., Aadhaar not 12 digits, invalid PAN format, or mobile number not 10 digits), a relevant error message will be returned.
    - **`/getUserDetails/{customerId}` (GET):**
        - Retrieves the job role and monthly income of a customer based on their `customerId`.
        - **Response:**
          ```json
          {
            "jobRole": "Salaried",
            "monthlyIncome": 5000
          }
          ```

### **ADMIN Capabilities:**
- `ADMIN` can view details of all registered `CUSTOMER` users.
- `ADMIN` can filter users based on registration date range, salary limit, and job role.

### **Endpoints for ADMIN:**

#### **`/admin/getAllUsers` (GET):**
- **Description:**
    - Allows an `ADMIN` to fetch the details of all registered users.
- **Response:**
  ```json
  [
    {
      "id": "customer1",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "aadhaarNumber": "123456789012",
      "panCardNumber": "ABCDE1234F",
      "address": "123 Main St, City, Country",
      "mobileNumber": "9876543210",
      "jobRole": "Salaried",
      "monthlyIncome": 5000,
      "registrationDate": "2023-08-10T14:48:00.000Z"
    },
    {
      "id": "customer2",
      "name": "Jane Doe",
      "email": "jane.doe@example.com",
      "aadhaarNumber": "123456789013",
      "panCardNumber": "ABCDE1234G",
      "address": "456 Market St, City, Country",
      "mobileNumber": "9876543211",
      "jobRole": "Freelancer",
      "monthlyIncome": 10000,
      "registrationDate": "2023-09-12T11:20:00.000Z"
    }
  ]
  ```

#### **`/admin/filterUsers` (GET):**
- **Description:**
    - Allows an `ADMIN` to filter users based on a registration date range, job role, and monthly income.
- **Query Parameters:**
    - `startDate`, `endDate` (optional): Filters based on registration dates.
    - `minSalary`, `maxSalary` (optional): Filters based on monthly income.
    - `jobRole` (optional): Filters by job role.

## **2. Credit Card Service (Merged Card Issuance & Billing):**
- **Functionalities:**
    - Issue new credit cards to customers based on their eligibility.
    - Assign an initial credit limit based on predefined rules.
    - Generate monthly bills for each customer based on their transactions.
    - Allow customers to view their current and past bills.
- **Entities:**
    - **`Card`:**
        - Represents the credit card details.
        - **Attributes:**
            - `cardId`: Unique identifier for the card.
            - `customerId`: Identifier for the associated customer.
            - `cardNumber`: Encrypted 16-digit card number (validated to be 16 digits before encryption).
            - `cvv`: Encrypted 3-digit CVV (validated to be 3 digits).
            - `expirationDate`: Card expiration date (validated to be a future date in MM/YY format).
            - `creditLimit`: The maximum credit limit assigned to the card.
            - `outstandingBalance`: The current outstanding balance for the card (total unpaid amount).
            - `active`: Boolean to indicate whether the card is active.
        - **Business Logic:**
            - When issuing a card, the system should check the customer's eligibility based on their job role and monthly income. If eligible, the card is created with an initial credit limit, and the `active` attribute is set to `true`.

- **Initial Credit Limit:**
  The initial credit limit is based on the customer’s monthly income and job role

| **Job Role** | **Monthly Income**  | **Initial Credit Limit** |
|-------|---------------------|--------------------------|
| Salaried | < 20,000            | 5,000 - 10,000           |
| Salaried | 20,000 - 50,000     | 10,000 - 50,000          |
| Salaried | > 50,000            | 50,000 - 100,000         |
| Freelancer | < 20,000            | 2,000 - 5,000            |
| Freelancer | > 50,000            | 25,000 - 60,000          |
| Other | < 20,000            | 50,00 - 10,000           |
| Other | 20,000 - 50,000     | 10,000 - 50,000          |
| Other | > 50,000            | 50,000 - 100,000         |


- **Endpoints:**
    - **`/issue` (POST):**
        - Issues a new credit card to a registered customer. This should be initiated by an `ADMIN`.
        - **Request Body:**
          ```json
          {
            "customerId": "123456"
          }
          ```
        - **Response:** Returns the details of the issued credit card, including:
            - `cardNumber`, `creditLimit`, `expirationDate`, `cvv`
        - **Logic for Credit Limit Assignment:**
            - The credit limit will be determined based on predefined business rules that take into account the customer’s job role and monthly income, which are retrieved from the User Registration and Authentication Service internally using the `customerId`.

    - **`/getCard/{customerId}` (GET):**
        - Retrieves the credit card details of the customer based on the `customerId`, including a check if the card is active. If the card is inactive, an appropriate message should be returned.

    - **`/generateBill/{customerId}` (POST):**
        - Generates a new monthly bill for a customer based on their transactions.
    - **`/getBill/{customerId}` (GET):**
        - Retrieves the current or past bills for the customer.

---

## **3. Payments Service:**
- **Functionalities:**
    - Allow customers to make payments towards their credit card bills.
    - Update the outstanding balance when a payment is made.
    - Allow customers to use their credit card for payments to merchants.
- **Entities:**
    - **`Payment`:**
        - Stores payment information.
        - **Attributes:**
            - `paymentId`: Unique identifier for the payment.
            - `amount`: Amount paid (validated to be a positive number).
            - `date`: Date of payment.
            - `paymentStatus`: Status of the payment (Completed/Pending).
    - **`Transaction`:**
        - Represents a transaction made using the credit card.
        - **Attributes:**
            - `transactionId`: Unique identifier for the transaction.
            - `customerId`: Identifier for the associated customer.
            - `merchantId`: Identifier for the merchant receiving the payment.
            - `amount`: Amount paid to the merchant (validated to be a positive number).
            - `date`: Date of transaction.
            - `transactionStatus`: Status of the transaction (Success/Failed).

- **Endpoints:**
    - **`/makePayment/{customerId}` (POST):**
        - Allows a customer to make a payment towards their bill.
        - Upon successful payment processing, a synchronous call should be made to the Credit Card Service to update the card balance.
        - **Request Body:**
          ```json
          {
            "amount": 200
          }
          ```
        - **Response:** Confirms the payment and updates the outstanding balance, including a status message.

    - **`/payToMerchant` (POST):**
        - Allows a customer to make a payment to a merchant using their credit card.
        - **Request Body:**
          ```json
          {
            "customerId": "123456",
            "merchantId": "merchant-001",
            "amount": 150,
            "cardNumber": "1234-5678-9012-3456",
            "cvv": "123",
            "expirationDate": "12/25"
          }
          ```
        - **Validation:** Before processing the payment, the system will validate the card details (e.g., check if the card is active, if the CVV is correct, and if the expiration date is valid).
        - **Response:** Confirms the transaction and updates the customer's card balance accordingly, including a transaction ID and status message.

- **Payment Transaction Failure Handling:**
    - In case of a payment failure (e.g., insufficient funds, expired card, incorrect CVV, or network issues), the system should:
        - Log the failure with a reason.
        - Return a failure response with an appropriate error message (e.g., "Insufficient funds" or "Invalid CVV").
- Allow the customer to retry the payment.
- Ensure idempotency for retrying failed payments, so the same transaction is not processed multiple times.
- If the failure is due to temporary issues (e.g., network failure), retry the payment up to three times before returning an error to the user.

---

## **4. Spring Cloud Gateway:**
- **Routing:** Acts as the entry point for all incoming requests and routes them to the appropriate microservice.


---

## **5. Eureka Service Discovery:**
- **Service Registration and Discovery:** All microservices should register with Eureka to enable dynamic service discovery and load balancing.

---

## **Inter-Microservice Communication:**
- The following endpoints will be used for synchronous inter-service communication:
    - **From Credit Card Service to User Registration Service:**
        - **`/getUserDetails/{customerId}` (GET):**
            - Retrieves user details, specifically job role and monthly income, from the User Registration and Authentication Service based on the `customerId` to determine eligibility for credit card issuance.

    - **From Payments Service to Credit Card Service:**
        - **`/getCard/{customerId}` (GET):**
            - Retrieves the credit card details from the Credit Card Service to validate the card status and details before processing a payment.

    - **From Payments Service to Credit Card Service (on successful payment):**
        - **`/updateCardBalance/{customerId}` (PUT):**
            - Updates the card balance after a payment is made. This will include reducing the balance by the payment amount.

---

### Non-Functional Requirement

- Ensure MySQL is used as the database to store data.
- Issue of new card has to be logged in a CSV file named card_issue.csv using Java NIO.
- New payment logs has to be saved in a CSV file named payment.csv using Java NIO.
- Apply thread safety and synchronization wherever required.
- Input Validation: Validate all inputs thoroughly.
- Exception Handling: Handle invalid transactions gracefully.
- Use a logger to record all system errors for auditing purposes.

-----
### **API Documentation:**
The project must also include detailed API documentation, covering all endpoints, request/response structures, validation rules, and error handling for each service.
