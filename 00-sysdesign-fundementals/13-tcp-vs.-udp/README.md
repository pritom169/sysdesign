## TCP vs. UDP


Think of the internet as a massive system of mail delivery. **TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)** are the two primary methods used to transport data packets between computers. They both run on top of the IP (Internet Protocol) layer, but they have completely different philosophies.

- **TCP** is like a **Certified Mail** service: It guarantees your letter arrives, in order, and sends you a receipt.
- **UDP** is like sending a **Postcard**: You drop it in the mailbox and hope it gets there. It's fast and cheap, but there is no tracking number.

Here is a detailed breakdown of how each works.

---

### 1. TCP (Transmission Control Protocol)

TCP is designed for **reliability**. It ensures that no matter how chaotic the network is, the data received is exactly the same as the data sent.

#### **Phase A: The Connection (The 3-Way Handshake)**

Before any data is transferred, TCP establishes a "virtual circuit" between two devices. This is known as the **3-way handshake**.

1. **SYN (Synchronize):** The client sends a packet with the `SYN` flag on. This is effectively asking, "Hello, I want to connect. My sequence number is ."
2. **SYN-ACK (Synchronize-Acknowledge):** The server receives it and replies with `SYN-ACK`. It says, "I received your request (). I am ready too. My sequence number is ."
3. **ACK (Acknowledge):** The client replies with `ACK`. It says, "I received your reply (). Let's start."

- _Result:_ A stateful connection is established.

#### **Phase B: Reliable Data Transfer**

Once connected, TCP manages the data flow using three sophisticated mechanisms:

- **Sequencing:** TCP chops data into segments and assigns a **Sequence Number** to each. If packets arrive out of order (e.g., packet 2 arrives after packet 3), the receiver uses these numbers to reassemble them correctly.
- **Acknowledgments (ACKs) & Retransmission:** When the receiver gets a packet, it sends an ACK back to the sender.
- _The "What if" scenario:_ If the sender does not receive an ACK within a specific timeframe (timeout), it assumes the packet was lost and **retransmits** it automatically.

- **Flow Control (Windowing):** The receiver tells the sender how much buffer space it has left (called the "Window Size"). If the receiver is overwhelmed, the sender slows down. This prevents the sender from drowning the receiver in data.

#### **Phase C: Connection Termination**

When the data transfer is done, TCP politely closes the connection using a **4-way handshake**:

1. **FIN:** Sender says "I'm done."
2. **ACK:** Receiver says "Received."
3. **FIN:** Receiver says "I'm done too."
4. **ACK:** Sender says "Goodbye."

---

### 2. UDP (User Datagram Protocol)

UDP is designed for **speed**. It strips away all the "safety checks" of TCP to reduce latency (delay).

#### **How It Works (Fire-and-Forget)**

UDP does **not** establish a connection (no handshake). It simply takes the application data, adds a tiny header, and throws it onto the network.

- **No Guarantees:** UDP does not care if the packet arrives. It does not check for errors (mostly) and does not ask for acknowledgments.
- **No Ordering:** If you send packets A, B, and C, they might arrive as C, A, B. The application layer must figure out what to do with them.
- **Header Simplicity:** Because it lacks these features, the UDP header is incredibly small—only **8 bytes**. In contrast, the TCP header is **20-60 bytes**.

---

### HTTP: 1.0 vs 1.1 vs 2.0 vs 3.0

#### HTTP 1.0

**How it worked:** It introduced Headers, allowing the transfer of images, video, and scripts, not just HTML. It also added status codes (like 404 Not Found).

**Limitation:** It still required opening a new TCP connection for every single file. If a website had 10 images, your browser had to set up and tear down 10 separate connections, which was incredibly slow.

#### HTTP 1.1

Released in 1997, this version powered the web for over 15 years and is still widely used.

**Key Innovation:** Persistent Connections ("Keep-Alive") Instead of opening a new connection for every file, HTTP/1.1 keeps the TCP connection open to reuse it for multiple requests (e.g., HTML, CSS, and images all travel over one wire).

**The Critical Flaw:** Head-of-Line Blocking HTTP/1.1 processes requests sequentially. If your browser requests a large image followed by a small script, the small script has to wait for the large image to finish downloading completely. It is like a single checkout lane at a grocery store; if the person in front of you has a full cart, you have to wait, even if you only have one item.

#### HTTP 2.0

Released in 2015, HTTP/2 was a massive redesign intended to fix the "checkout lane" problem of HTTP/1.1.

How it works:

- **Binary Framing:** It creates a binary layer separate from the visible text headers. It breaks data down into tiny binary "frames" of 1s and 0s, making it easier for machines to parse.

- **Multiplexing:** This is the game-changer. It allows multiple requests and responses to be sent at the same time over a single connection. The browser can request the HTML, CSS, and JS simultaneously, and the server sends them back in mixed chunks.

- **Analogy:** Instead of one checkout lane where customers wait in line (HTTP/1.1), HTTP/2 is like a conveyor belt where everyone puts their items on at once, and they get sorted out at the end.

- **The Remaining Flaw:** HTTP/2 still relies on TCP (Transmission Control Protocol). If a single packet of data gets lost in the network, TCP forces all streams to stop and wait until that packet is re-transmitted. This is called TCP Head-of-Line Blocking.

#### HTTP 3.0

Standardized recently (2022), HTTP/3 abandons TCP entirely to solve the packet-loss issue.

- **How it works (QUIC Protocol):** Instead of TCP, HTTP/3 uses a new protocol called QUIC, which is built on top of UDP (User Datagram Protocol).

- **Independent Streams:** Because it uses UDP, packet loss only affects the specific stream it belongs to. If a packet for an image is lost, the CSS and JavaScript streams continue loading without pause.

- **Faster Handshakes:** It combines the connection setup and encryption (TLS) handshake into one step (or even zero steps for repeat visitors), making connections establish much faster.

