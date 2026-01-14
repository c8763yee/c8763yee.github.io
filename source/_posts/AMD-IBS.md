---
title: AMD IBS

---

# AMD IBS
:::danger
# TODO:
- 整理格式
- 檢查 MSR 中Found in Linux 的部分與了解為什麼文章[^1]沒提到
- 發Patch修註解(arch/x86/include/asm/amd/ibs.h)
- 寫一個可以循序訪問Physical memory range 的workload
:::
## 定義[^1]
IBS 是一個基於硬體, 用來收集 Fetch/OP 階段中的 instruction 資訊的 sampling method
## 功能
### IBS Fetch sampling
IBS Hardware 會在 instruction fetch 階段從 Memory/Cache 隨機選取 fetch block (包含 Instruction Bytes 的資料)來收集資訊

Tag 的 Fetch 結束後，IBS 硬體會將該 Block 的資訊寫入 IBS Fetch Control Register， Virtual/Physical Address 分別寫入至 IBS Fetch Linear Address Register 和 IBS Fetch Physical Address Register，並設定 IbsFetchVal = 1，同時送出中斷給本地 APIC，讓APIC 來通知OS有新的 Fetch Sample 可讀取，並將 IbsFetchVal 清除為 0，等待下一次 Sample

可設定/獲得的資訊:

1. If the fetch completed or was aborted (fetch 行為是否完成)
2. The number of core **clock cycles** spent on the fetch (本次 Fetch 所花費的 Cycle 數)
3. If the fetch hit or missed the instruction cache (Cache Hit/Miss)
4. If the instruction fetch hit or missed the instruction TLBs(ITLB Hit/Miss)
5. The fetch address translation page size (Fetch address 轉換的 Page Size, 4KB/2MB/1GB)
6. The **linear and physical address** associated with the fetch(block address)
### IBS execution (OP) sampling
Instruction 會被分成多個 Micro Operations(uOps, op)，IBS會在固定Cycle/分配Ops數量(based on IbsOpCntCtl bit)後 tag 一個 op, 該 op 會跟其他 op 一起等待被分配或執行，並在該 tagged op retired（op對應的 Instruction 執行完成）時才會回傳資料(Q：回傳/提醒方式?)

回傳資料時，會將Tagged Op 的執行資訊寫入 IBS Execution Register(IbsOpData{1..3})，並設定IbsOpVal然後傳送中斷到 local APIC 

可獲取的資訊
> - **Branch status** for branch ops.(Hit/Miss)
> - 對於 load/store op:
>   - Whether the load/store missed in the data cache(**cache miss**).
>   - Whether the load/store address hit or missed in the TLBs (**TLB miss**).
>   - The **linear and physical address** of the data operand associated with  the load/store operation.
>   - **Source information** for cache, DRAM, MMIO, or I/O accesses.

## Sampling 機制
- Fetch: 透過 IbsFetchMaxCnt 調整，當 內部20-bit Counter 的 19:4 Bit 等於IbsFetchMaxCnt時 Tag下一個 Fetch Block，Fetch 結束後 Counter 的 lower 4-bit 會根據 IbsRandEn設為隨機數或歸零
- Op: 透過 IbsOpMaxCnt 調整，當 內部 27-bit Counter 的 26:4 bit 等於 IbsOpMaxCnt 時 Tag 下一個 Op，Tagged Op retired 後 Counter 的 lower 4-bit 會根據該tagged op 是否 retired 歸零26:7 bit 與隨機設定6:0 bit
## 過濾 sample 
- 當sample 不符合 filter 條件時，硬體丟棄該Sample並重置Counter
    - Fetch: 將Counter 的 lower 6-bit設為隨機或規零（based on IbsRandEn），Higher bit歸零
    - Op: 將 Counter 歸零, 並設定lower 6-bit設為隨機，
- 目前AMD64 Architecture Programmer's Manual文檔中僅提到 L3 Cache miss 可過濾[^1]
### Trigger 模式差異
Cycle: 當CPU Counter 經過指定的 Cycle 次數後，抓取下一個資料(Fetch Block/Op)，並重新計數
Event: 在IBS將 Retired Op 視為 Event, 經過指定Op數後抓取下一個Op，並重新計數
## 支援性(CPUID)[^2]
- 8000_0001h: Extended Processor and Processor Feature Identifiers
ECX[IBS] (10) bit: 檢查CPU是否支援IBS
- 8000_001Bh: Instruction-Based Sampling Capabilities
EAX[IbsL3MissFiltering] (11) bit: 檢查CPU 是否能只抓l3missonly

## IBS MSR
透過 RDMSR/WRMSR 指令來讀寫 IBS 相關的 MSR
### IBS Fetch Control Register(IbsFetchCtl)
![image](https://hackmd.io/_uploads/r1CcNuR4Wx.png)
用來同時儲存資訊與設定 Sampling 行為
- Found in Linux kernel[^3]
    58 bit(fetch_l2_miss): L2 miss for sampled fetch(needs IbsFetchComp) 
    56 bit(l2tlb_miss): i-cache fetch missed in L2TLB 

### IBS Fetch Linear Address Register(IbsFetchLinAd)

Read-only, 儲存 64-Bit 的 Fetch Instruction Virtual Address
只在 IbsFetchCtl[IbsFetchVal] (49) bit 同時設為 1 時有意義
![image](https://hackmd.io/_uploads/ryR7Du0NZx.png)

### IBS Fetch Physical Address Register (IbsFetchPhysAd)

Read-Only，儲存 52-Bit 的 Fetch Instruction Physical Address
52Bit 只是架構限制，各 Processer 實際能存取的大小可能更小
只在 IbsFetchCtl[IbsPhyAddrValid] (52) 跟 IbsFetchCtl[IbsFetchVal] (49) bit 同時設為 1 時有意義
![image](https://hackmd.io/_uploads/HJqhwuAN-x.png)


### IBS Execution Control Register (IbsOpCtl)

![image](https://hackmd.io/_uploads/rJCGeLXH-g.png)
- Found in Linux kernel[^3]
    32-58 Bit(opcurcnt): periodic op counter current count
    59-62 Bit(ldlat_thrsh): Load Latency threshold
    63 Bit(ldlat_en): Load Latency enabled

### IBS Op Linear Address Register (IbsOpRip)
![image](https://hackmd.io/_uploads/rydJbImr-g.png)
Read-only, 儲存 64-Bit Tagged op 對應到的 Instruction 的 Virtual Address
IbsOpCtl[IbsOpVal]bit =1 與 IbsOpData1[IbsRipInvalid] =1時有意義

### IBS Op Data 1 Register (IbsOpData1)
![image](https://hackmd.io/_uploads/H1sIiU7SWg.png)
Op狀態相關數據
- Found in Linux Kernel
    0-15 Bit(comp_to_ret_ctr): op completion to retire count
    16-31 Bit(tag_to_ret_ctr): op tag to retire count
    39 Bit(op_brn_fuse): fused branch op
    40 Bit(op_microcode): microcode op

### IBS Op Data 2 Register (IbsOpData2)
NorthBridge 相關數據

```c

/*MSR 0xc0011036: IBS Op Data 2
 * |reserved:56|data_src_hi:1|cache_hit_st:1|rmt_node:1|
 *                              reserved:1|data_src_lo:3|
 *
 */
union ibs_op_data2 {
	__u64 val;
	struct {
		__u64	data_src_lo:3,	/* 0-2: data source low */
			reserved0:1,	/* 3: reserved */
			rmt_node:1,	/* 4: destination node */
			cache_hit_st:1,	/* 5: cache hit state */
			data_src_hi:2,	/* 6-7: data source high */
			reserved1:56;	/* 8-63: reserved */
	};
};
```
### IBS Op Data 3 Register (IbsOpData3)
紀錄 op 的 Cache相關數據，Unaligned Access讀取時只抓Lower part
- Found in Linux Kernel
    4 Bit(dc_l1tlb_hit_2m): data cache L1TLB hit in 2M page
    5 Bit(dc_l1tlb_hit_1g): data cache L1TLB hit in 1G page
    16 Bit(dc_miss_no_mab_alloc): DC miss with no MAB allocated
    19 Bit(dc_l2_tlb_hit_1g): data cache L2 hit in 1GB page
    20 Bit(l2_miss): L2 cache miss
    21 Bit(sw_pf): software prefetch
    22-25 Bit(op_mem_width): load/store size in bytes
    26-31 Bit(op_dc_miss_open_mem_reqs): outstanding mem reqs on DC fill
    48-63 Bit(tlb_refill_lat): L1 TLB refill latency
    
![image](https://hackmd.io/_uploads/BkZf8DXSbl.png)
![image](https://hackmd.io/_uploads/SkUGIDQS-g.png)

## Use with Perf[^4]
### 透過 IBS找出所有 load/store Operation
- IbsOpData3[IbsLdOP|IbsStOP]
```bash
sudo perf record -d  -e ibs_op// --phys-data -a -c 100000 -- ./workload
# workload: dd if=/dev/zero of=/dev/null bs=4K count=1K

sudo perf script -F pid,tid,cpu,ip,addr,phys_addr,data_src,comm | grep -E 'OP (LOAD|STORE)'
```

---

# Ref

[^1]: [AMD64 Architecture Programmer’s Manual, Volume 2: System Programming](https://docs.amd.com/api/khub/documents/sD1_QL~h4Afq2_tvzxqqSQ/content?Ft-Calling-App=ft%2Fturnkey-portal&Ft-Calling-App-Version=5.2.22#page=486&zoom=100,84,238)
[^2]: [AMD64 Architecture Programmer's Manual Volume 3: General Purpose and System Programming Instructions (PUB) (24594)](https://docs.amd.com/v/u/en-US/24594_3.37)
[^3]: [Linux Kernel v6.18.3: arch/x86/include/asm/amd/ibs.h](https://elixir.bootlin.com/linux/v6.18.3/source/arch/x86/include/asm/amd/ibs.h)
[^4]: [man perf-amd-ibs(1)](https://man.archlinux.org/man/perf-amd-ibs.1.en)
<!-- 
[^2]: [Software Optimization Guide for the AMD Zen5 Microarchitecture](https://docs.amd.com/v/u/en-US/58455_1.00) -->