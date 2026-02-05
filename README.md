# MATF-RVTECH-AWSCLOUD-2025


Redosled komandi koje se pokrecu kada se kod preuzme na drugoj masini.
Sa zvezdicom oznaceni one koje se uvek pokrecu, ostale se pokrecu samo prvi put.

----
Potrebni alati:
- docker-compose
- localstack
- awslocal
---

1. Proveriti ima li postojeceg kontejnera:

`docker-compose ps`

---
2. * Zaustavi postojece kontejnere ako postoje:

`docker-compose down -v`              

Ako ne radi moze samo docker compose

---
3. * Pokreni LocalStack:

`docker-compose up -d`

---
4. Instaliraj serverless dependencies

`npm install`

---
5. * Deploy serverless na localstack:

`npx serverless deploy`

---
6. Zipuj lambda funkcije i ubaci u kontejner:

`zip function.zip index.js`

`docker cp function.zip rvtech-matf-2025-localstack-chargers:/function.zip`

---
7. Prekopiraj izgled sajta u kontejner:

`docker cp web rvtech-matf-2025-localstack-chargers:/web`

---
8. **Dalji koraci se rade iz  kontejnera sa localstackom / preko awscli-local**

Za kontejner sa localstackom:
`docker exec -it rvtech-matf-2025-localstack-chargers bash`

Za awscli-local:
Napraviti venv ako ne postoji
`python -m venv rivianvenv`

Instalirati awscli-local:
`pip install awscli-local`

Aktivirati venv
`source ./rivianvenv/bin/activate`


---
9. Kreiranje lambda funkcija:



 ```
 awslocal lambda create-function \
  --function-name sync-ocm-data \
  --runtime nodejs18.x \
  --handler index.syncOCMdata \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --zip-file fileb://function.zip \
  --environment "Variables={OCM_API_KEY=69d86e58-df21-406b-b837-cc07c35ddd41,OCM_URL=https://api.openchargemap.io/v3/poi,CHARGERS_TABLE=Chargers}"  
  ```
  
  ```
  
  awslocal lambda create-function \
  --function-name get-chargers-by-town \
  --runtime nodejs18.x \
  --handler index.getChargersByTown \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --zip-file fileb://function.zip \
  --environment "Variables={CHARGERS_TABLE=Chargers}"
```

---
10. Napravi s3 bucket:

`awslocal s3 mb s3://chargers-website`

---
11. Sync s3 bucket:

`awslocal s3 sync /web s3://chargers-website`

---
12. * Pronalazak api_id i azuriranje u index.html:

`awslocal apigateway get-rest-apis --query 'items[0].id'`

"""
140        // KONFIGURACIJA - promeni API_ID ako treba
141        // Dobij sa: awslocal apigateway get-rest-apis --query 'items[0].id' --output text
142        const API_ID = '4lm8cpkcvk';
143        const API_BASE_URL = `http://localhost:4566/restapis/${API_ID}/dev/_user_request_`;
"""

---
13. * Deploy frontend na S3:

`npm run deploy-frontend-fixed-bucket`

Sajt se nalazi na jednoj od ove dve adrese, u zavisnosti od mesta pokretanjai:
- http://chargers-website.s3-website.localhost.localstack.cloud:4566/
- http://localhost:4566/chargers-website/index.html
