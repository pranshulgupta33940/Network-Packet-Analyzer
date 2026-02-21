# 🚀 60-Day DPI Engine Study Plan (30 Mins/Day)

This plan is designed to take you from a basic understanding of C++ to deeply understanding how network packets are parsed, tracked, and blocked in a highly concurrent environment. Spend strictly **30 minutes a day** to avoid burnout and maximize retention.

---

## 📅 Month 1: The Single-Threaded Core (Weeks 1-4)
*Goal: Understand exactly how a packet is read, parsed, and classified before introducing multi-threading.*

### Week 1: Networking Basics & PCAP Files
- **Day 1-2:** Read the "Networking Background" section of the `README.md`. Understand the 5-Tuple and the layers (MAC -> IP -> TCP -> Payload).
- **Day 3:** Look at `include/types.h`. Understand the `FiveTuple` struct and `AppType` enum.
- **Day 4:** Read `include/pcap_reader.h`. What is a PCAP Global Header versus a Packet Header?
- **Day 5-6:** Read `src/pcap_reader.cpp`. Trace how the code opens a file and iterates through the bytes to pull out individual packets.
- **Day 7:** Review Week 1. Open `test_dpi.pcap` in Wireshark (if installed) just to see what a packet visualizer looks like.

### Week 2: Peeling the Onion (Packet Parsing)
- **Day 8:** Look at `include/packet_parser.h`. Note the `ParsedPacket` struct.
- **Day 9:** Read `parseEthernet` in `src/packet_parser.cpp`. Focus on how it skips 14 bytes to get to the IP payload.
- **Day 10-11:** Read `parseIPv4` in `src/packet_parser.cpp`. Look up what "Endianness" (big-endian vs little-endian) means and why `ntohs()` is used.
- **Day 12-13:** Read `parseTCP` and `parseUDP` in `src/packet_parser.cpp`. How does it find the source and destination ports?
- **Day 14:** Review Week 2. You now know how to extract the 5-Tuple!

### Week 3: Deep Packet Inspection (SNI Extraction)
- **Day 15:** Read "How SNI Extraction Works" in the `README.md`. Understand the TLS Client Hello message.
- **Day 16:** Look at `include/sni_extractor.h`. Notice we have both TLS (HTTPS) and HTTP extractors.
- **Day 17-18:** Read the `extract` function in `src/sni_extractor.cpp` for TLS. Trace how it skips the session ID, cipher suites, and looks for the `0x0000` extension.
- **Day 19-20:** Read the HTTP extraction logic in `src/sni_extractor.cpp`. How does it find the "Host:" header?
- **Day 21:** Review Week 3. You now know how to spy on encrypted traffic destinations!

### Week 4: Tying it Together (Single-Threaded Engine)
- **Day 22-23:** Read `src/main_working.cpp`. Start from `main()`. See how it uses `PcapReader`.
- **Day 24:** Trace the loop in `main_working.cpp` where it calls `PacketParser::parse()`.
- **Day 25:** Understand how the `flows` hash map works in `main_working.cpp`. Why are we storing state per connection?
- **Day 26:** Trace the SNI extraction and rule blocking logic in the main loop.
- **Day 27-28:** Review Month 1. Write down the complete journey of a packet on a piece of paper from memory.

---

## 📅 Month 2: Mastering High Performance Concurrency (Weeks 5-8)
*Goal: Understand the producer-consumer architecture, lock-free queues, and consistent hashing that makes the engine fast.*

### Week 5: Multi-Threading Primitives
- **Day 29-30:** Read about `std::thread`, `std::mutex`, and `std::condition_variable` online (C++ reference or tutorials).
- **Day 31:** Look at `include/thread_safe_queue.h`.
- **Day 32-33:** Study the `push()` and `pop()` methods in the thread-safe queue. Why does `pop()` wait on a condition variable instead of using a `while(true)` loop (busy-waiting)?
- **Day 34-35:** Read `include/dpi_engine.h` and `src/dpi_engine.cpp`. How does it initialize the rules and statistics?

### Week 6: The Load Balancer & Consistent Hashing
- **Day 36-37:** Read `include/load_balancer.h` and `src/load_balancer.cpp`.
- **Day 38-39:** Understand the `hash(tuple) % num_fps` logic in the load balancer. Why must the same 5-tuple always go to the same Fast Path thread? (Hint: Consistent Hashing).
- **Day 40-41:** Read `include/connection_tracker.h` and `src/connection_tracker.cpp`. How does it manage memory for old, dead connections? (LRU/Timeout logic).
- **Day 42:** Review Week 6. You now understand how work is divided safely!

### Week 7: The Fast Path (Worker Threads)
- **Day 43-44:** Read `include/fast_path.h` and `src/fast_path.cpp`.
- **Day 45-46:** Trace the `run()` loop in `fast_path.cpp`. See how it pops from its input queue, looks up the connection state, and performs DPI.
- **Day 47-48:** Read `include/rule_manager.h` and `src/rule_manager.cpp`. How fast is it to check if a domain is blocked?
- **Day 49:** Review Week 7. You now know how the actual heavy lifting is done in parallel.

### Week 8: The Grand Finale (The Main MT Engine)
- **Day 50-51:** Open `src/dpi_mt.cpp`. Start from `main()`.
- **Day 52-53:** Trace how `dpi_mt.cpp` creates the Load Balancers and Fast Path threads, and how it wires their queues together.
- **Day 54-55:** Trace the main thread (the Reader). See how it pushes packets into the Load Balancers' queues.
- **Day 56:** Understand the output writer thread in `dpi_mt.cpp`. How does it collect packets from all FPs and write them back to disk?
- **Day 57-58:** Compile and run the code! Test changing the rules and thread counts.
- **Day 59:** Review Month 2. Explain the entire multi-threaded architecture out loud to an imaginary interviewer.
- **Day 60:** **CELEBRATE!** 🎉 You are now officially a systems engineer!

---

## 💡 Pro-Tips for your 30-Minute Sessions:
1. **Don't just read; Write comments:** When you read a file, add your own `//` comments explaining what a block of code does in your own words.
2. **Follow the data flow:** Don't read a file top-to-bottom. Start at `main()` and jump to functions as they are called.
3. **If you get stuck on syntax:** Spend 5 minutes googling the specific C++ feature (e.g., "what is std::optional in C++"). Do not let syntax stop you from understanding the logic.
