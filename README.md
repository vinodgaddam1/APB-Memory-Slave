# APB-Memory-Slave

## 📌 1. Project Title
✅ **APB (AMBA) Memory-Mapped Slave — Parameterized RAM with PREADY**

---

## 📝 2. Description
This project implements a simple **APB (Advanced Peripheral Bus) slave** that exposes a **memory array** (RAM) to an APB master. It supports:

- ✍️ **APB Write** → stores `PWDATA` into `mem[PADDR]`
- 📖 **APB Read** → outputs `mem[PADDR]` on `PRDATA`
- ✅ Handshake using **`PSELx` + `PENABLE`** and response **`PREADY`**

The design is **parameterized** for width and depth, and includes an **active-low reset** (`PRESETn`) that clears memory.

---

## ⭐ 3. Key Features
- 🚌 APB-style interface signals (`PCLK, PRESETn, PADDR, PWDATA, PRDATA, PWRITE, PSELx, PENABLE, PREADY`)
- ⚙️ Parameterized RAM:
  - `WIDTH` (default **8-bit**)
  - `DEPTH` (default **16 locations**)
  - `ADDR = $clog2(DEPTH)` address bits
- ✅ `PREADY` asserted when a valid APB access occurs (`PSELx && PENABLE`)
- 🔄 Reset clears `PRDATA`, deasserts `PREADY`, and initializes memory to zero

---

## 🔌 4. Module Interface

### 📄 Module Declaration
```verilog
module apb(PCLK,PRESETn,PADDR,PWDATA,PRDATA,PWRITE,PSELx,PREADY,PENABLE);
```

### 🧾 Ports
| Signal     | Dir | Width | Description |
|-----------|-----|-------|-------------|
| `PCLK`    | In  | 1     | ⏱️ APB clock |
| `PRESETn` | In  | 1     | 🔄 Active-low reset |
| `PADDR`   | In  | `ADDR`| 📍 Address |
| `PWDATA`  | In  | `WIDTH` | 📨 Write data |
| `PRDATA`  | Out | `WIDTH` | 📤 Read data |
| `PWRITE`  | In  | 1     | ✍️ 1=Write, 0=Read |
| `PSELx`   | In  | 1     | 🎯 Peripheral select |
| `PENABLE` | In  | 1     | ✅ Access enable phase |
| `PREADY`  | Out | 1     | 🤝 Transfer ready response |

---

## 🧠 5. Functional Behavior (APB Transaction Model)

### 🔄 Reset Behavior (`PRESETn = 0`)
- `PRDATA <= 0`
- `PREADY <= 0`
- 🧼 All memory locations cleared to `0`

### ✅ Valid APB Access
An APB transfer is considered active when:
```verilog
PSELx && PENABLE
```

When active:
- `PREADY <= 1` ✅ (indicates slave response ready)

#### ✍️ Write Transfer (`PWRITE = 1`)
```verilog
mem[PADDR] <= PWDATA;
```

#### 📖 Read Transfer (`PWRITE = 0`)
```verilog
PRDATA <= mem[PADDR];
```

### 🚫 No Transfer
If `PSELx && PENABLE` is not true:
- `PREADY <= 0`

---

## ⚙️ 6. Parameter Example
Example: build a **32 x 16-bit** APB memory:
```verilog
apb #(
  .WIDTH(16),
  .DEPTH(32)
) u_apb_mem (
  .PCLK(PCLK),
  .PRESETn(PRESETn),
  .PADDR(PADDR),
  .PWDATA(PWDATA),
  .PRDATA(PRDATA),
  .PWRITE(PWRITE),
  .PSELx(PSELx),
  .PREADY(PREADY),
  .PENABLE(PENABLE)
);
```

---

## 📁 7. Files
- 📄 `apb.v` — APB memory slave module

