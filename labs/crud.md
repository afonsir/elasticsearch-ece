```json
POST bank/_doc/1000
{
  "account_number" : 1000,
  "balance" : 65536,
  "firstname" : "John",
  "lastname" : "Doe",
  "age" : 23,
  "gender" : "M",
  "address" : "125 Bear Creek Pkwy",
  "employer" : "Linux Academy",
  "email" : "john@linuxacademy.com",
  "city" : "Keller",
  "state" : "TX"
}
```

```json
POST bank/_update/100
{
  "doc": {
    "address" : "1600 Pennsylvania Ave NW",
    "city" : "Washington",
    "state" : "DC"
  }
}
```

```json
DELETE bank/_doc/1

DELETE bank/_doc/10
```