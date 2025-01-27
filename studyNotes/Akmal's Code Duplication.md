# Akmal's Code Duplication

###### tags: `2024`

**Goal:**
- [x] Compile and Run Akmal's Thesis Code

**References:**
- [Akmal Handover | Thesis Repository User Guide](https://hackmd.io/@akmalns/BJKrRdqua#A-Installation)

1. Ubuntu version = 18.04.6
![image](https://hackmd.io/_uploads/HyuWCzV26.png)

2. Linux Machine = 64 bit
![image](https://hackmd.io/_uploads/r1dB0MEhT.png)

3. Clone Directory `git clone https://github.com/bmw-ece-ntust/MIMO-Scheduler-on-Sch-Slice-Based.git`

4. Branch Checkout `git checkout o1-perf-metrics`

5. Compile O-DU `make odu MACHINE=BIT64 MODE=FDD`
![image](https://hackmd.io/_uploads/ByVAoL4np.png)

6. Compile cu_stub `make cu_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD`
![image](https://hackmd.io/_uploads/r1KjjI43p.png)

7. Compile ric_stub `make ric_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD`
![image](https://hackmd.io/_uploads/BJtln8V3T.png)

8. ifconfig (this is the standard IP)
![image](https://hackmd.io/_uploads/rkR-B44n6.png)

9. Run [Video](https://youtu.be/sHgKra5WHWI)