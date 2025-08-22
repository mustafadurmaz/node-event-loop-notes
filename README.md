# node-event-loop-notes

### 1️⃣ Basic — Sync, Microtask, Timeout, Immediate, I/O

const fs = require("fs");

console.log("A - sync");

setTimeout(() => console.log("timeout 0"), 0);
setImmediate(() => console.log("immediate"));

Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));

fs.readFile(__filename, () => {
  console.log("readFile done");
});

console.log("B - sync");

### Beklenen çıktı:

A - sync
B - sync
nextTick
promise
timeout 0
immediate
readFile done

### 2️⃣ I/O İçinden setTimeout vs setImmediate

const fs = require("fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout in IO"), 0);
  setImmediate(() => console.log("immediate in IO"));
});

### Beklenen çıktı:

immediate in IO
timeout in IO

### 3️⃣ CPU-bound Bloklama

console.log("start");

const end = Date.now() + 3000; // 3 saniye CPU yoğun iş
while (Date.now() < end) {}    // Event Loop BLOKLANDI

setTimeout(() => console.log("timeout after block"), 0);

console.log("end")

### Beklenen çıktı:

start
end
timeout after block

### 4️⃣ HTTP İsteği (I/O Yoğun)

const https = require("https");

console.log("before request");

https.get("https://jsonplaceholder.typicode.com/todos/1", (res) => {
  res.on("data", () => {});
  res.on("end", () => {
    console.log("http request done");
  });
});

console.log("after request");

### Beklenen çıktı:

before request
after request
http request done

### 5️⃣ Veritabanı Simülasyonu (Async I/O)

function fakeDbQuery(query, cb) {
  setTimeout(() => {
    cb(null, `Result of ${query}`);
  }, 1000);
}

console.log("querying db...");
fakeDbQuery("SELECT * FROM users", (err, res) => {
  console.log("db result:", res);
});
console.log("doing other work...");

### Beklenen çıktı:

querying db...
doing other work...
db result: Result of SELECT * FROM users

### 6️⃣ CPU + I/O Karma Senaryo

const fs = require("fs");

console.log("start");

setTimeout(() => console.log("timeout"), 0);

fs.readFile(__filename, () => {
  console.log("I/O done");
});

Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));

const end = Date.now() + 2000;
while (Date.now() < end) {} // CPU bloklama

console.log("end");

### Beklenen çıktı:

start
end
nextTick
promise
timeout
I/O done
