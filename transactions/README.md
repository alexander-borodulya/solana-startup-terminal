**Практика до лекції з транзакціями**

---

1. Відправка SOL
- Згенерувати ключі отримувача
- Надіслати йому SOL
- Перевірити статус транзакції

2. Виклик Memo
- Зчитати поточні priority fees
- Підвищити CU price + CU limit
- Відправити транзакцію Memo program
- Перевірити статус транзакції

3. Використання nonce account

---
**Надсилання з Rust (1, 2 завдання)**

 Збілдіть і запустіть проект
```bash
cargo run -p transactions
```

---
**Використання nonce account з cli**

Ціль - підписання транзакції декількома учасниками

1. Згенеруйте учасників

```bash
solana-keygen new -o sender.json
solana-keygen new -o co-sender.json
solana-keygen new -o receiver.json
```

2. Airdrop

```bash
solana airdrop -k sender.json 0.5
solana airdrop -k co-sender.json 0.5
```

3. Cтягнемо останній блокхеш:

```bash
curl https://api.devnet.solana.com -s -X \
  POST -H "Content-Type: application/json" -d ' 
  {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "getLatestBlockhash",
    "params": [
      {
        "commitment": "processed"
      }
    ]
  }
' 
```

4. 1 підпис

```bash
solana transfer receiver.json 0.1 \
  --sign-only \
  --blockhash <blockhash> \
  --fee-payer co-sender.json \
  --from <sender-pubkey> \
  --keypair co-sender.json
```

5. 2 підпис

```bash
solana transfer receiver.json 0.1 \
  --allow-unfunded-recipient \
  --blockhash <blockhash> \
  --from sender.json \
  --keypair sender.json \
  --signer <signers-from-resp>
```

6. Error: Hash has expired 😿

7. Створіть ключі під нонс акаунт та nonce authority

```bash
solana-keygen new -o nonce-account.json
solana-keygen new -o nonce-authority.json
```

8. Airdrop на authority
```bash
solana airdrop -k nonce-authority.json 0.5
```

8. Створіть nonce account
```bash
solana create-nonce-account nonce-account.json 0.0015
```

9. Зчитайте дані
```bash
solana nonce-account nonce-account.json
```

10. Використайте nonce blockhash в транзакції трансферу - 1 підпис
```bash
solana transfer receiver.json 0.1 \
  --sign-only \
  --nonce nonce-account.json \
  --blockhash <nonce-blockhash> \
  --fee-payer co-sender.json \
  --from <sender-pubkey> \
  --keypair co-sender.json
```

11. Використайте nonce blockhash в транзакції трансферу - 2 підпис
```bash
solana transfer receiver.json 0.1 --allow-unfunded-recipient \
  --nonce nonce-account.json \
  --nonce-authority nonce-authority.json \
  --blockhash <nonce-blockhash> \
  --from sender.json \
  --keypair sender.json \
  --signer <signers-from-resp>
  ```