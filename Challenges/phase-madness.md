# Phase Madness — HTB Challenge Writeup

**Type:** Challenge | **Difficulty:** Easy | **Category:** Quantum

---

## Summary

Phase Madness is a quantum-computing challenge that exposes an interactive quantum circuit oracle over TCP. The server encodes each byte of the flag as a rotation angle (in degrees) on an independent qubit, then lets the player inject extra gates and request measurement statistics. Because the qubits share no entanglement, the encoding is a classical angle-recovery problem dressed in quantum clothing: repeated measurement across 100,000 shots reveals each qubit's Bloch-vector components, from which the original byte is reconstructed with basic trigonometry.

---

## Enumeration / Analysis

The challenge zip contains a single file: `server.py`. Reading it reveals everything needed to plan the attack before touching the live instance.

**Circuit construction (`PhaseEncoder`):**

```python
for i in range(0, len(flag_bytes), 3):
    theta = flag_bytes[i]          # degrees
    qc.rx(math.radians(theta), i)

    if i+1 < n:
        theta = flag_bytes[i+1]
        qc.ry(math.radians(theta), i+1)

    if i+2 < n:
        theta = flag_bytes[i+2]
        qc.h(i+2)
        qc.rz(math.radians(theta), i+2)
```

No two-qubit (entangling) gates are applied anywhere. Each qubit's final state depends solely on its corresponding flag byte.

**Interactive protocol (one round):**

1. Server prompts: `Specify the qubit index you want to measure :`
2. Server prompts: `Specify the instructions :` — accepts a `;`-separated list of `GATE:phase,qubit` (only `RX`, `RY`, `RZ` are injectable; `H` and `CX` are not).
3. Server responds with JSON measurement counts from 100,000 shots, e.g. `{"0": 73214, "1": 26786}`.

**Flag length probe:** Querying a deliberately out-of-range qubit index like `10000` yields an error message containing the circuit size:

```
Index 10000 out of range for size 79
```

Flag length confirmed: **79 bytes**. Re-probe on every fresh instance spawn since flag content (and thus byte count) may differ.

---

## Exploitation

### Bloch-vector analysis

With no entanglement, each qubit starts in |0⟩ and is rotated by exactly one base gate. Working out the Bloch-vector `(X, Y, Z)` for each base-gate pattern at angle θ (degrees):

| qubit index mod 3 | base gate(s) | Bloch vector |
|---|---|---|
| 0 | `RX(θ)` | `(0, −sin θ, cos θ)` |
| 1 | `RY(θ)` | `(sin θ, 0, cos θ)` |
| 2 | `H; RZ(θ)` | `(cos θ, sin θ, 0)` |

A standard Z-basis measurement returns `|1⟩` with probability `P(1) = (1 − Z) / 2`.

### RX / RY qubits — single baseline query

For `i % 3 ∈ {0, 1}`, the Z-component is `cos θ`, so no extra gate is needed:

```
P(1) = (1 − cos θ) / 2
cos θ = 1 − 2·P(1)
θ = degrees(acos(1 − 2·P(1)))
```

`acos` returns a value in `[0°, 180°]`. Because printable ASCII is `[32, 126]`, the mirror root `360° − θ` would always land above 234° — never a valid flag byte — so the result is unambiguous without further disambiguation.

### H;RZ qubits — two queries

For `i % 3 == 2`, the Z-component is always 0 regardless of θ, so a bare measurement returns ~50/50 and carries no information. Two queries with injected gates recover the XY components:

**Query 1** — inject `RY:90,i`:

```
P(1) = (1 + cos θ) / 2  →  cos θ = 2·P(1) − 1
```

**Query 2** — inject `RX:90,i`:

```
P(1) = (1 − sin θ) / 2  →  sin θ = 1 − 2·P(1)
```

Then:

```python
theta = math.degrees(math.atan2(sin_theta, cos_theta)) % 360
byte_val = round(theta)
```

### Solver

Total queries: 1 per RX/RY qubit + 2 per H;RZ qubit ≈ 132 round trips for a 79-byte flag.

```python
import socket, json, math, sys

def query(sock, qubit_idx, instructions=""):
    # Send qubit index
    sock.sendall(f"{qubit_idx}\n".encode())
    recv_until_prompt(sock)        # read "Specify the instructions :"
    sock.sendall(f"{instructions}\n".encode())
    raw = recv_until_prompt(sock)  # read JSON counts line
    counts = json.loads(extract_json(raw))
    shots = sum(counts.values())
    return counts.get("1", 0) / shots   # P(1)

def recover_byte(sock, i):
    kind = i % 3
    if kind in (0, 1):                  # RX / RY base — baseline only
        p1 = query(sock, i, "")
        cos_t = 1 - 2 * p1
        cos_t = max(-1.0, min(1.0, cos_t))
        theta = math.degrees(math.acos(cos_t))
    else:                               # H;RZ base — two cross-axis queries
        p_ry = query(sock, i, f"RY:90,{i}")
        p_rx = query(sock, i, f"RX:90,{i}")
        cos_t = max(-1.0, min(1.0, 2 * p_ry - 1))
        sin_t = max(-1.0, min(1.0, 1 - 2 * p_rx))
        theta = math.degrees(math.atan2(sin_t, cos_t)) % 360
    return round(theta) % 256
```

Run against the live instance:

```bash
python3 solve.py <target-ip> <port>
```

The script prints decoded bytes as it goes. If the server drops the connection mid-run (a known quirk of this instance), reconnect and resume from where it left off — each byte query is independent.

**Recovered flag:** `HTB{...}`

---

## Key Takeaways

- **Quantum state tomography as a classical side-channel:** when qubits are independent, the "quantum" wrapper is irrelevant — each qubit encodes one scalar angle recoverable by standard tomography.
- **Gate-type awareness matters:** the optimal measurement basis depends on which base gate was applied (`RX`/`RY` vs `H;RZ`). Choosing the wrong extra gate (or skipping it when unnecessary) wastes queries or introduces ambiguity.
- **Printable ASCII range eliminates `acos` ambiguity:** the mirror root of `acos` always falls outside `[0, 127]`, making single-query decoding unambiguous for RX/RY qubits without extra angle-disambiguation logic.
- **Tools:** Python 3, `qiskit`, `qiskit-aer` (for local circuit validation), raw TCP socket client.