# Flowchart sources (Mermaid)

Paste these into any Mermaid-aware tool (GitHub, Notion, Confluence, Excalidraw,
Gamma, mermaid.live) to regenerate the diagrams used in `index.html`.

---

## 1. System architecture (slide 4)

```mermaid
flowchart LR
    A["Implant Source<br/>pay.cpp · pay_linux.c"] --> B["Builder<br/>inject host/port · compile"]
    B --> C["Compiled Implant<br/>build_output/payload.*"]
    C -->|beacon| D["TCP Transport<br/>:7777 · XOR+hex / CC20"]
    D --> E["Control Plane<br/>Flask APIs · listeners"]
    F["AI Operator<br/>agent · tool calls"] -->|tool calls| E
    E -->|results / beacon| F
```

## 2. Protocol round trip (slide 7)

```mermaid
flowchart LR
    cmd["cmd<br/>plaintext"] --> xor1["XOR<br/>key 0x3A"]
    xor1 --> hex1["hex<br/>encode"]
    hex1 --> tcp["TCP<br/>newline-delimited"]
    tcp --> hex2["hex<br/>decode"]
    hex2 --> xor2["XOR<br/>key 0x3A"]
    xor2 --> exec["exec<br/>on host"]
    exec --> resp["response<br/>re-encode"]
    resp -.->|XOR 0x3A -> hex -> TCP| tcp
```

## 3. Agent operator loop (slide 10)

```mermaid
flowchart TD
    obj["Natural-language objective"] --> plan["Planner / LLM"]
    plan --> guard["Scope guardrail<br/>allow-list · denylist · approval"]
    guard --> tools["Tool calls<br/>build · listen · command"]
    tools --> parse["Output parser<br/>triage · reason trace"]
    parse -->|re-plan| plan
```

## 4. Guardrail decision flow (slide 12)

```mermaid
flowchart LR
    act["Agent action"] --> q1{"target in allow-list?"}
    q1 -->|no| block["Block + log"]
    q1 -->|yes| q2{"action in denylist?"}
    q2 -->|yes| block
    q2 -->|no| q3{"destructive?"}
    q3 -->|yes| appr["Human approval"]
    appr --> exec["Execute + audit log"]
    q3 -->|no| exec
```

## 5. Demo workflow (slide 13)

```mermaid
flowchart LR
    s1["1 · Objective<br/>natural language"] --> s2["2 · Build implant<br/>select + compile"]
    s2 --> s3["3 · Listener up<br/>register port"]
    s3 --> s4["4 · Command target<br/>!sysinfo / uname"]
    s4 --> s5["5 · Triage output<br/>decide next action"]
    s5 -.->|parse, re-plan, next tool| s1
```

---

## Demo run workflow (shell)

Build-time pipeline for the live demo, as a shell sequence:

```bash
# 1. control plane (Web UI :5000, C2 listener :7777)
python3 web_listener.py

# 2. Windows implant
x86_64-w64-mingw32-windres resources.rc -o resources.o
x86_64-w64-mingw32-g++ pay.cpp resources.o -o build_output/payload.exe \
  -lws2_32 -liphlpapi -lcrypt32 -lpsapi -lgdi32 -luser32 -s -O2 -mwindows

# 3. Linux implant
gcc -o build_output/payload.elf pay_linux.c -lX11 -lpthread -lcrypt -ldl -lm -s -O2
```
