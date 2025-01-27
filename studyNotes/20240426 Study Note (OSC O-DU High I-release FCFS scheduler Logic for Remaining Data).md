# 2024/04/26 Study Note (OSC O-DU High I-release FCFS scheduler Logic for Remaining Data)

###### tags: `2024`


**Goal:**
- [x] [Check OSC's FCFS logic (what happens if UE1 have remaining data that was unallocated in first TTI)](#0-Summary)


**References:**
- [2024/04/15 Study Note (OSC O-DU High I-release FCFS scheduler with RRM Policy Implementation)](https://github.com/bmw-ece-ntust/guideline-template/blob/wilfridAzariah/studyNotes/20240415%20Study%20Note%20(OSC%20O-DU%20High%20I-release%20FCFS%20scheduler%20with%20RRM%20Policy%20Implementation).md)


**Outline:**
- [2024/04/26 Study Note (OSC O-DU High I-release FCFS scheduler Logic for Remaining Data)](#2024-04-26-study-note--osc-o-du-high-i-release-fcfs-scheduler-logic-for-remaining-data-)
          + [tags: `2024`](#tags---2024-)
  * [0. Summary](#0-summary)
  * [1. Objective of this Note](#1-objective-of-this-note)
  * [2. Prediction](#2-prediction)
  * [3. Analysis Steps for UE Deletion from `ueToBeScheduled`](#3-analysis-steps-for-ue-deletion-from--uetobescheduled-)
    + [3.1. When does a UE gets deleted from or reordered in `ueToBeScheduled` list?](#31-when-does-a-ue-gets-deleted-from-or-reordered-in--uetobescheduled--list-)
    + [3.2. When will `isDlMsgScheduled` be TRUE?](#32-when-will--isdlmsgscheduled--be-true-)
    + [3.3. When will `accumalatedSize` == 0?](#33-when-will--accumalatedsize-----0-)
    + [3.4. Conclusion](#34-conclusion)
  * [4. Extra Code Trace: Where does UE get added into `ueToBeScheduled`?](#4-extra-code-trace--where-does-ue-get-added-into--uetobescheduled--)
  * [5. Example RUN](#5-example-run)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>



## 0. Summary

- Assume `ueToBeScheduled` List contains UE1 and UE2 in order

![image](https://hackmd.io/_uploads/HkXiEfJGC.png)

- When UE1 is scheduled in this SLOT but have remaining data that was unallocated in this SLOT, the remaining data will be scheduled after UE2. Not necessarly after all UE2 data is allocated (similar to Round Robin). For details, see steps for UE deletion from `ueToBeScheduled` in [conclusion](#34-Conclusion)

![image](https://hackmd.io/_uploads/HyI04f1zA.png)


## 1. Objective of this Note

- Based on Wilfrid's and Bimo's discussion on [2024/04/09](https://hackmd.io/@superwilfrid/rk9If1F56#20240409) about Multi UE Scheduling for 1 Slice, there are some analysis that are needed for OSC's FCFS Multi UE scheduling:
    1. Explain the root cause of weird "OSC RRMPolicy" implementation
    2. Check the validity of the example [use case](https://hackmd.io/@superwilfrid/r1TKoOP10#03-Compare)
    3. Check OSC's FCFS logic (what happens if UE1 have remaining data that was unallocated in first TTI)

- This note is made to answer the 3rd question.


![image](https://hackmd.io/_uploads/ryx-322gC.png)



## 2. Prediction

- Assume that PRB per TTI is less and UE1 LC 3-2 does not get PRB. Will it be allocated first in next TTI, or move to the back of UE 2 LC 3-2?
- There is 2 options to answer number 3:
    1. UE1 LC 3-2 allocated first in next TTI before UE 2 LC 1-1
    2. UE1 LC 3-2 allocated move to the back of UE 2 LC 3-2

## 3. Analysis Steps for UE Deletion from `ueToBeScheduled`

### 3.1. When does a UE gets deleted from or reordered in `ueToBeScheduled` list?

- `schFcfsScheduleSlot` full code (`~/src/5gnrsch/sch_fcfs.c`):

```c=1130
/*******************************************************************
 *
 * @brief Scheduling of Slots in UL And DL 
 *
 * @details
 *
 *    Function : schFcfsScheduleSlot
 *
 *    Functionality: Scheduling of slots in UL and DL specific to 
 *       FCFS scheduling
 *
 * @params[in] Pointer to Cell
 *             Slot timing info
 *             Scheduler instance
 * @return void
 *
 * ****************************************************************/
void schFcfsScheduleSlot(SchCellCb *cell, SlotTimingInfo *slotInd, Inst schInst)
{
   SchFcfsCellCb  *fcfsCell;
   SchFcfsUeCb    *fcfsUeCb;
   SchDlHqProcCb  *hqP = NULLP;
   SchUlHqProcCb  *ulHqP = NULLP;
   CmLList        *pendingUeNode;
   CmLList        *node;
   uint8_t        ueId, ueCount = 0;
   bool           isRarPending = false, isRarScheduled = false;
   bool           isMsg4Pending = false, isMsg4Scheduled = false;
   bool           isDlMsgPending = false, isDlMsgScheduled = false;
   bool           isUlGrantPending = false, isUlGrantScheduled = false;

   fcfsCell = (SchFcfsCellCb *)cell->schSpcCell;

   /* Select first UE in the linked list to be scheduled next */
   pendingUeNode = fcfsCell->ueToBeScheduled.first;
   ueCount = fcfsCell->ueToBeScheduled.count;

   while(pendingUeNode && ueCount > 0)
   {
      /*Since Multi-UE perTTI is not supported, re-init following parameters.*/
      isRarPending = false; isRarScheduled = false;
      isMsg4Pending = false; isMsg4Scheduled = false;
      isDlMsgPending = false; isDlMsgScheduled = false;
      isUlGrantPending = false; isUlGrantScheduled = false;
      if(pendingUeNode->node)
      {
         ueId = *(uint8_t *)(pendingUeNode->node);
         fcfsUeCb = (SchFcfsUeCb *)cell->ueCb[ueId-1].schSpcUeCb;

         /* If RAR is pending for this UE, schedule PDCCH,PDSCH to send RAR and 
          * PUSCH to receive MSG3 as per k0-k2 configuration*/
         if(cell->raReq[ueId-1] != NULLP)
         {
            isRarPending = true;
            isRarScheduled = schProcessRaReq(schInst, cell, *slotInd, ueId);
         }

         /*MSG3 retransmisson*/
         if(cell->raCb[ueId-1].retxMsg3HqProc)
         {            
            schMsg3RetxSchedulingForUe(&(cell->raCb[ueId-1]));
         }

         /* If MSG4 is pending for this UE, schedule PDCCH,PDSCH to send MSG4 and
          * PUCCH to receive UL msg as per k0-k1 configuration  */
         if (cell->ueCb[ueId-1].retxMsg4HqProc) //should work from dlmap later tbd
         {
            /* Retransmission of MSG4 */
            isMsg4Pending = true;
            if(schProcessMsg4Req(cell, *slotInd, ueId, TRUE, &cell->ueCb[ueId-1].retxMsg4HqProc) == ROK)
               isMsg4Scheduled = true;
         }
         else
         {
            /* First transmission of MSG4 */
            if(cell->raCb[ueId-1].msg4recvd)
            {
               isMsg4Pending = true;
               if(schProcessMsg4Req(cell, *slotInd, ueId, FALSE, &cell->ueCb[ueId-1].msg4HqProc) == ROK)
                  isMsg4Scheduled = true;

               /* If MSG4 scheduling failed, free the newly assigned HARQ process */
               if(!isMsg4Scheduled)
                  schDlReleaseHqProcess(cell->ueCb[ueId-1].msg4HqProc);
            }
         }

         if(isRarPending || isMsg4Pending)
         {
            /* If RAR or MSG is successfully scheduled then
             * remove UE from linked list since no pending msgs for this UE */
            if(isRarScheduled || isMsg4Scheduled)
            {
               schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
            }
            /* If RAR/MSG4 is pending but couldnt be scheduled then,
             * put this UE at the end of linked list to be scheduled later */
            else
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }

#ifdef NR_DRX 
         if((cell->ueCb[ueId-1].ueDrxInfoPres == true) && (cell->ueCb[ueId-1].drxUeCb.drxDlUeActiveStatus != true))
         {
            if(pendingUeNode->node)
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }
         else 
#endif
         {

            /* DL Data */
            node = NULLP;
            if(fcfsUeCb)
               node = fcfsUeCb->hqRetxCb.dlRetxHqList.first;
            if(node != NULLP)
            {
               /* DL Data ReTransmisson */
               isDlMsgPending = true;
               isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, TRUE, ((SchDlHqProcCb**) &(node->node)));
               if(isDlMsgScheduled)
               {
#ifdef NR_DRX 
                  schDrxStopDlHqRetxTmr(cell, &cell->ueCb[ueId-1], ((SchDlHqProcCb**) &(node->node)));
#endif
                  schFcfsRemoveFrmDlHqRetxList(&cell->ueCb[ueId-1], node);
               }
            }
            else
            {
               /* DL Data new transmission */
               if((cell->boIndBitMap) & (1<<ueId))
               {
                  isDlMsgPending = true;               
                  isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, FALSE, &hqP);

                  /* If DL scheduling failed, free the newly assigned HARQ process */
                  if(!isDlMsgScheduled)
                     schDlReleaseHqProcess(hqP);
                  else
                  {
#ifdef NR_DRX
                     schHdlDrxInActvStrtTmr(cell, &cell->ueCb[ueId-1], gConfigInfo.gPhyDeltaDl + SCHED_DELTA);
#endif
                  }
               }
            }
         }
#ifdef NR_DRX 
         if((cell->ueCb[ueId-1].ueDrxInfoPres == true) && (cell->ueCb[ueId-1].drxUeCb.drxUlUeActiveStatus != true))
         {
            if(pendingUeNode->node)
            {
               cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
            }
         }
         else 
#endif
         {
            /* Scheduling of UL grant */
            node = NULLP;
            if(fcfsUeCb)
               node = fcfsUeCb->hqRetxCb.ulRetxHqList.first;
            if(node != NULLP)
            {
               /* UL Data ReTransmisson */
               isUlGrantPending = true;
               isUlGrantScheduled = schProcessSrOrBsrReq(cell, *slotInd, ueId, TRUE, (SchUlHqProcCb**) &(node->node));
               if(isUlGrantScheduled)
               {
#ifdef NR_DRX 
                  schDrxStopUlHqRetxTmr(cell, &cell->ueCb[ueId-1], ((SchUlHqProcCb**) &(node->node)));
#endif
                  schFcfsRemoveFrmUlHqRetxList(&cell->ueCb[ueId-1], node);
               }
            }
            else
            {
               /* UL Data new transmission */
               if(cell->ueCb[ueId-1].srRcvd || cell->ueCb[ueId-1].bsrRcvd)
               {
                  isUlGrantPending = true;
                  isUlGrantScheduled = schProcessSrOrBsrReq(cell, *slotInd, ueId, FALSE, &ulHqP);
                  if(!isUlGrantScheduled)
                     schUlReleaseHqProcess(ulHqP, FALSE);
                  else
                  {
#ifdef NR_DRX
                     schHdlDrxInActvStrtTmr(cell, &cell->ueCb[ueId-1], gConfigInfo.gPhyDeltaUl + SCHED_DELTA);
#endif
                  }
               }
            }

            if(isUlGrantPending || isDlMsgPending)
            {
               if((isUlGrantPending && !isUlGrantScheduled) || (isDlMsgPending && !isDlMsgScheduled))
               {
                  cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
               }
               else
               {
                  schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
               }
            }
         }
      }
      if(!isUlGrantPending && !isDlMsgPending && !isRarPending && !isMsg4Pending)
      {
         DU_LOG("\nERROR  -->  SCH: In SchFcfsScheduleSlot, UE:%d is wrongly queued\
               in Pending UE List without any actions, Removing the UE from the list",ueId);
         schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
      }
      if(cell->schDlSlotInfo[slotInd->slot]->prbAlloc.numPrbAlloc >= MAX_NUM_RB)
      {
         DU_LOG("\nINFO   -->  SCH: No PRB available to proceed with next UE");
         return;     
      }
      pendingUeNode = fcfsCell->ueToBeScheduled.first;
      ueCount--;
   }
}
```


- UE will be deleted from `ueToBeScheduled` list if isDlMsgpending and isDlMsgScheduled for the UE are both TRUE. If only isDlMsgpending is TRUE, UE will be put into the end of the link list to be rescheduled again next time
```c=1328
if(isUlGrantPending || isDlMsgPending)
{
    if((isUlGrantPending && !isUlGrantScheduled) || (isDlMsgPending && !isDlMsgScheduled))
    {
        cmLListAdd2Tail(&fcfsCell->ueToBeScheduled, cmLListDelFrm(&fcfsCell->ueToBeScheduled, pendingUeNode));
    }
    else
    {
        schFcfsRemoveUeFrmScheduleLst(cell, pendingUeNode);
    }
}
```

- Another line of code to focus on is if number of PRB allocation of the particular slot is >= `MAX_NUM_RB`, then `schFcfsScheduleSlot` will return without changing the order.
```c=1347
if(cell->schDlSlotInfo[slotInd->slot]->prbAlloc.numPrbAlloc >= MAX_NUM_RB)
{
    DU_LOG("\nINFO   -->  SCH: No PRB available to proceed with next UE");
    return;     
}
```


- Key takeaway:
    - If DL MSG scheduled, the current UE will be removed from `ueToBeScheduled` List
    - If DL MSG is not scheduled, the current UE will be moved to the back of `ueToBeScheduled` List
    - In each slot, the first UE will be prioritized. Only if `numPrbAlloc < MAX_NUM_RB`, then the next UE will be examined.


### 3.2. When will `isDlMsgScheduled` be TRUE?

- `isDlMsgScheduled` is a return variable of `schFillBoGrantDlSchedInfo`
```c=1267
isDlMsgPending = true;               
isDlMsgScheduled = schFillBoGrantDlSchedInfo(cell, *slotInd, ueId, FALSE, &hqP);
```


- `schFillBoGrantDlSchedInfo` full code (`~/src/5gnrsch/sch_slot_ind.c`):

```c=76
/*******************************************************************
 *
 * @brief 
 *
 * @details
 *
 *    Function : schFillBoGrantDlSchedInfo 
 *
 *    Functionality:
 
 *
 * @params[in] SchCellCb *cell, SlotTimingInfo currTime, uint8_t ueId
 * @params[in] bool isRetx, SchDlHqProcCb **hqP
 * @return ROK     - success
 *         RFAILED - failure
 *
 * ****************************************************************/
bool schFillBoGrantDlSchedInfo(SchCellCb *cell, SlotTimingInfo currTime, uint8_t ueId, bool isRetx, SchDlHqProcCb **hqP)
{
   uint8_t pdschNumSymbols = 0, pdschStartSymbol = 0;
   uint8_t lcIdx = 0;
   uint16_t startPrb = 0;
   uint16_t crnti = 0;
   uint32_t accumalatedSize = 0;
   SchUeCb *ueCb = NULLP;
   DlMsgSchInfo *dciSlotAlloc, *dlMsgAlloc;
   SlotTimingInfo pdcchTime, pdschTime, pucchTime;
   SchPdcchAllocInfo pdcchAllocInfo;

   GET_CRNTI(crnti,ueId);
   ueCb = &cell->ueCb[ueId-1];

   if (isRetx == FALSE)
   {
      if(schDlGetAvlHqProcess(cell, ueCb, hqP) != ROK)
      {
         return false;
      }
   }

   memset(&pdcchAllocInfo,0,sizeof(SchPdcchAllocInfo));
   if(findValidK0K1Value(cell, currTime, ueId, ueCb->k0K1TblPrsnt,\
            &pdschStartSymbol, &pdschNumSymbols, &pdcchTime, &pdschTime, \
            &pucchTime, isRetx, *hqP, &pdcchAllocInfo) != true )
   {
      /* If a valid combination of slots to scheduled PDCCH, PDSCH and PUCCH is
       * not found, do not perform resource allocation. Return from here. */
      return false;
   }
   
   /* allocate PDCCH and PDSCH resources for the ue */
   if(cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId-1] == NULL)
   {

      SCH_ALLOC(dciSlotAlloc, sizeof(DlMsgSchInfo));
      if(!dciSlotAlloc)
      {
         DU_LOG("\nERROR  -->  SCH : Memory Allocation failed for ded DL msg alloc");
         return false;
      }
      cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId -1] = dciSlotAlloc;
      memset(dciSlotAlloc, 0, sizeof(DlMsgSchInfo));
   }
   else
   {
      dciSlotAlloc = cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId -1];
   }
   /* Dl ded Msg info is copied, this was earlier filled in macSchDlRlcBoInfo */
   fillDlMsgInfo(dciSlotAlloc, crnti, isRetx, *hqP);
   dciSlotAlloc->transportBlock[0].ndi = isRetx;

   accumalatedSize = cell->api->SchScheduleDlLc(pdcchTime, pdschTime, pdschNumSymbols, &startPrb, isRetx, hqP);

   /*Below case will hit if NO LC(s) are allocated due to resource crunch*/
   if (!accumalatedSize)
      return false;

   /*[Step6]: pdcch and pdsch data is filled */
   if((schDlRsrcAllocDlMsg(cell, pdschTime, crnti, accumalatedSize, dciSlotAlloc, startPrb,\
                           pdschStartSymbol, pdschNumSymbols, isRetx, *hqP, pdcchAllocInfo)) != ROK)
   {
      DU_LOG("\nERROR  --> SCH : Scheduling of DL dedicated message failed");

      /* Free the dl ded msg info allocated in macSchDlRlcBoInfo */
      if(!dciSlotAlloc->dlMsgPdschCfg)
      {
         SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
         cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId -1] = NULL;
      }
      return false;
   }

   /* TODO : Update the scheduling byte report for multiple LC based on QCI
    * and Priority */
   /* As of now, the total number of bytes scheduled for a slot is divided
    * equally amongst all LC with pending data. This is avoid starving of any
    * LC 
    * */
#if 0
   accumalatedSize = accumalatedSize/dlMsgAlloc->numLc;
   for(lcIdx = 0; lcIdx < dlMsgAlloc->numLc; lcIdx ++)
      dlMsgAlloc->lcSchInfo[lcIdx].schBytes = accumalatedSize;
#endif
   
   /* Check if both DCI and DL_MSG are sent in the same slot.
    * If not, allocate memory for DL_MSG PDSCH slot to store PDSCH info */

   if(pdcchTime.slot == pdschTime.slot)
   {
      SCH_ALLOC(dciSlotAlloc->dlMsgPdschCfg, sizeof(PdschCfg));
      if(!dciSlotAlloc->dlMsgPdschCfg)
      {
         DU_LOG("\nERROR  -->  SCH : Memory Allocation failed for dciSlotAlloc->dlMsgPdschCfg");
         SCH_FREE(dciSlotAlloc->dlMsgPdcchCfg, sizeof(PdcchCfg));
         SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
         cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId-1] = NULLP;
         return false;
      }
      memcpy(dciSlotAlloc->dlMsgPdschCfg,\
         &dciSlotAlloc->dlMsgPdcchCfg->dci[dciSlotAlloc->dlMsgPdcchCfg->numDlDci - 1].pdschCfg,  sizeof(PdschCfg));
   }
   else
   {
      /* Allocate memory to schedule dlMsgAlloc to send DL_Msg, pointer will be checked at schProcessSlotInd() */
      if(cell->schDlSlotInfo[pdschTime.slot]->dlMsgAlloc[ueId-1] == NULLP)
      {
         SCH_ALLOC(dlMsgAlloc, sizeof(DlMsgSchInfo));
         if(dlMsgAlloc == NULLP)
         {
            DU_LOG("\nERROR  -->  SCH : Memory Allocation failed for dlMsgAlloc");
            SCH_FREE(dciSlotAlloc->dlMsgPdcchCfg, sizeof(PdcchCfg));
            if(dciSlotAlloc->dlMsgPdschCfg == NULLP)
            {
               SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
               cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId-1] = NULLP;
            }
            return false;
         }
         cell->schDlSlotInfo[pdschTime.slot]->dlMsgAlloc[ueId-1] = dlMsgAlloc;
         memset(dlMsgAlloc, 0, sizeof(DlMsgSchInfo));
      }
      else
         dlMsgAlloc = cell->schDlSlotInfo[pdschTime.slot]->dlMsgAlloc[ueId-1];

      /* Copy all DL_MSG info */
      dlMsgAlloc->crnti =crnti;
      dlMsgAlloc->bwp = dciSlotAlloc->bwp;
      SCH_ALLOC(dlMsgAlloc->dlMsgPdschCfg, sizeof(PdschCfg));
      if(dlMsgAlloc->dlMsgPdschCfg)
      {
         memcpy(dlMsgAlloc->dlMsgPdschCfg,\
                &dciSlotAlloc->dlMsgPdcchCfg->dci[dciSlotAlloc->dlMsgPdcchCfg->numDlDci - 1].pdschCfg, sizeof(PdschCfg));
      }
      else
      {
         SCH_FREE(dciSlotAlloc->dlMsgPdcchCfg, sizeof(PdcchCfg));    
         if(dciSlotAlloc->dlMsgPdschCfg == NULLP)
         {
            SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
            cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueId-1] = NULLP;

         }
         SCH_FREE(dlMsgAlloc, sizeof(DlMsgSchInfo));
         cell->schDlSlotInfo[pdschTime.slot]->dlMsgAlloc[ueId-1] = NULLP;
         DU_LOG("\nERROR  -->  SCH : Memory Allocation failed for dlMsgAlloc->dlMsgPdschCfg");
         return false;
      }
   }

   /*Re-setting the BO's of all DL LCs in this UE*/
   for(lcIdx = 0; lcIdx < MAX_NUM_LC; lcIdx++)
   {
      ueCb->dlInfo.dlLcCtxt[lcIdx].bo = 0;
   }

   /* after allocation is done, unset the bo bit for that ue */
   UNSET_ONE_BIT(ueId, cell->boIndBitMap);
   return true;
}
```


- First, we can read from the code that `isDlMsgScheduled` will return FALSE if there are failures in memory allocation. But that is not the main focus

- `isDlMsgScheduled` will return FALSE if `accumalatedSize ` == 0
```c=147
accumalatedSize = cell->api->SchScheduleDlLc(pdcchTime, pdschTime, pdschNumSymbols, &startPrb, isRetx, hqP);

/*Below case will hit if NO LC(s) are allocated due to resource crunch*/
if (!accumalatedSize)
    return false;
```

- Otherwise, `isDlMsgScheduled` will return TRUE
```c=251
/* after allocation is done, unset the bo bit for that ue */
UNSET_ONE_BIT(ueId, cell->boIndBitMap);
return true;
```


- Key takeaway:
    - As long as `accumalatedSize != 0`, `isDlMsgScheduled`/`schFillBoGrantDlSchedInfo` will be TRUE


### 3.3. When will `accumalatedSize` == 0?

- `accumalatedSize` is a return variable of `SchScheduleDlLc`. Default `SchScheduleDlLc` is **`schFcfsScheduleDlLc`**
```c=147
accumalatedSize = cell->api->SchScheduleDlLc(pdcchTime, pdschTime, pdschNumSymbols, &startPrb, isRetx, hqP);
```


- `schFcfsScheduleDlLc` full code (`~/src/5gnrsch/sch_fcfs.c`):

```c=963
/*******************************************************************
 *
 * @brief Grants resources to LC in downlink 
 *
 * @details
 *
 *    Function : schFcfsScheduleDlLc 
 *
 *    Functionality: Grants resources to LC in uplink
 *
 * @params[in] PDCCH Time
 *
 * @return ROK
 *         RFAILED
 *
 * ****************************************************************/
uint32_t schFcfsScheduleDlLc(SlotTimingInfo pdcchTime, SlotTimingInfo pdschTime, uint8_t pdschNumSymbols, \
                               uint16_t *startPrb, bool isRetx, SchDlHqProcCb **hqP)
{
   SchFcfsHqProcCb *fcfsHqProcCb;
   SchUeCb *ueCb;
   uint8_t lcIdx = 0;
   uint16_t maxFreePRB = 0;
   uint16_t mcsIdx = 0;
   uint32_t accumalatedSize = 0;
   CmLListCp *lcLL = NULLP;
   uint16_t rsvdDedicatedPRB = 0;
   DlMsgSchInfo *dciSlotAlloc;

   /* TX_PAYLOAD_HDR_LEN: Overhead which is to be Added once for any UE while estimating Accumulated TB Size
    * Following flag added to keep the record whether TX_PAYLOAD_HDR_LEN is added to the first Node getting allocated.
    * If both Dedicated and Default LC lists are present then First LC in Dedicated List will include this overhead
    * else if only Default list is present then first node in this List will add this overhead len*/
   bool isTxPayloadLenAdded = FALSE;

   ueCb = (*hqP)->hqEnt->ue;
   dciSlotAlloc = (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1];
   fcfsHqProcCb = (SchFcfsHqProcCb *)((*hqP)->schSpcDlHqProcCb);

   if (isRetx == FALSE)
   {
      /*Re-Initalization per UE*/
      /* scheduled LC data fill */
      dciSlotAlloc->transportBlock[0].numLc = 0;
      isTxPayloadLenAdded = FALSE; /*Re-initlaize the flag for every UE*/
      accumalatedSize = 0;

      for(lcIdx = 0; lcIdx < MAX_NUM_LC; lcIdx++)
      {
         if(ueCb->dlInfo.dlLcCtxt[lcIdx].bo)
         {
            /*Check the LC is Dedicated or default and accordingly LCList will
            * be used*/
            if(ueCb->dlInfo.dlLcCtxt[lcIdx].isDedicated)
            {
               lcLL = &(fcfsHqProcCb->lcCb.dedLcList);
               rsvdDedicatedPRB = ueCb->dlInfo.dlLcCtxt[lcIdx].rsvdDedicatedPRB;
            }
            else
            {
               lcLL = &(fcfsHqProcCb->lcCb.defLcList);
            }

            /*[Step2]: Update the reqPRB and Payloadsize for this LC in the appropriate List*/
            if(updateLcListReqPRB(lcLL, ueCb->dlInfo.dlLcCtxt[lcIdx].lcId,\
                     (ueCb->dlInfo.dlLcCtxt[lcIdx].bo + MAC_HDR_SIZE)) != ROK)
            {
               DU_LOG("\nERROR  --> SCH : Updation in LC List Failed");
               /* Free the dl ded msg info allocated in macSchDlRlcBoInfo */
               if(!dciSlotAlloc->dlMsgPdschCfg)
               {
                  SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
                  (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1] = NULL;
               }
               return accumalatedSize;
            }
         }
      }//End of for loop

      if ((fcfsHqProcCb->lcCb.defLcList.count == 0) && (fcfsHqProcCb->lcCb.dedLcList.count == 0))
      {
         DU_LOG("\nDEBUG  -->  SCH : No pending BO for any LC id\n");
         UNSET_ONE_BIT((*hqP)->hqEnt->ue->ueId, (*hqP)->hqEnt->cell->boIndBitMap);

         /* Free the dl ded msg info allocated in macSchDlRlcBoInfo */
         if(!dciSlotAlloc->dlMsgPdschCfg)
         {
            SCH_FREE(dciSlotAlloc, sizeof(DlMsgSchInfo));
            (*hqP)->hqEnt->cell->schDlSlotInfo[pdcchTime.slot]->dlMsgAlloc[ueCb->ueId -1] = NULL;
         }
         /*TRUE because this UE has nothing to be scheduled*/
         return accumalatedSize;
      }
   }

   /*[Step3]: Calculate Best FREE BLOCK with MAX PRB count*/
   maxFreePRB = searchLargestFreeBlock((*hqP)->hqEnt->cell, pdschTime, startPrb, DIR_DL);

   /*[Step4]: Estimation of PRB and BO which can be allocated to each LC in
    * the list based on RRM policy*/

   /*Either this UE contains no reservedPRB pool fir dedicated S-NSSAI or 
    * Num of Free PRB available is not enough to reserve Dedicated PRBs*/
   if(isRetx == FALSE)
   {
      if(maxFreePRB != 0)
      {
         mcsIdx = ueCb->ueCfg.dlModInfo.mcsIndex;

         if((fcfsHqProcCb->lcCb.dedLcList.count == NULLP) || ((maxFreePRB < rsvdDedicatedPRB)))
         { 
            fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB;
            DU_LOG("\nDEBUG  -->  SCH : DL Only Default Slice is scheduled, sharedPRB Count:%d",\
                  fcfsHqProcCb->lcCb.sharedNumPrb);

            /*PRB Alloc for Default LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols,\
                  &(fcfsHqProcCb->lcCb.sharedNumPrb), NULLP, &isTxPayloadLenAdded, NULLP);
         }
         else
         {
            fcfsHqProcCb->lcCb.sharedNumPrb = maxFreePRB - rsvdDedicatedPRB;
            /*PRB Alloc for Dedicated LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.dedLcList), TRUE, mcsIdx, pdschNumSymbols,\
                  &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);

            /*PRB Alloc for Default LCs*/
            prbAllocUsingRRMPolicy(&(fcfsHqProcCb->lcCb.defLcList), FALSE, mcsIdx, pdschNumSymbols, \
                  &(fcfsHqProcCb->lcCb.sharedNumPrb), &(rsvdDedicatedPRB), &isTxPayloadLenAdded, NULLP);
         }
      }
   }

   /*[Step5]:Traverse each LCID in LcList to calculate the exact Scheduled Bytes
    * using allocated BO per LC and Update dlMsgAlloc BO report for MAC */
   if (isRetx == FALSE)
   {
      if(fcfsHqProcCb->lcCb.dedLcList.count != 0)
         updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.dedLcList), dciSlotAlloc, NULLP, &(accumalatedSize));

      updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.defLcList), dciSlotAlloc, NULLP, &(accumalatedSize));
   }
   else
   {
      accumalatedSize = (*hqP)->tbInfo[0].tbSzReq;
   }

   /*Below case will hit if NO LC(s) are allocated due to resource crunch*/
   if (!accumalatedSize)
   {
      if(maxFreePRB == 0)
      {
         DU_LOG("\nERROR  --> SCH : NO FREE PRB!!");
      }
      else
      {
         /*Schedule the LC for next slot*/
         DU_LOG("\nDEBUG  -->  SCH : No LC has been scheduled");
      }
      /* Not Freeing dlMsgAlloc as ZERO BO REPORT to be sent to RLC so that
       * Allocation can be done in next slot*/
      accumalatedSize = 0;
   }

   return accumalatedSize;
}
```


- First, we can read from the code that `accumalatedSize` will be incremented based on the allocBO of each LC inside the `updateGrantSizeForBoRpt` function
```c=1096
/*[Step5]:Traverse each LCID in LcList to calculate the exact Scheduled Bytes
* using allocated BO per LC and Update dlMsgAlloc BO report for MAC */
if (isRetx == FALSE)
{
    if(fcfsHqProcCb->lcCb.dedLcList.count != 0)
        updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.dedLcList), dciSlotAlloc, NULLP, &(accumalatedSize));

    updateGrantSizeForBoRpt(&(fcfsHqProcCb->lcCb.defLcList), dciSlotAlloc, NULLP, &(accumalatedSize));
}
```

- `accumalatedSize` will return 0 if only no alloBO for all LC == 0
```c=1110
/*Below case will hit if NO LC(s) are allocated due to resource crunch*/
if (!accumalatedSize)
{
    if(maxFreePRB == 0)
    {
        DU_LOG("\nERROR  --> SCH : NO FREE PRB!!");
    }
    else
    {
        /*Schedule the LC for next slot*/
        DU_LOG("\nDEBUG  -->  SCH : No LC has been scheduled");
    }
    /* Not Freeing dlMsgAlloc as ZERO BO REPORT to be sent to RLC so that
    * Allocation can be done in next slot*/
    accumalatedSize = 0;
}

return accumalatedSize;
```


- Key takeaway:
    - As long as not all LC's allocBO == 0, `accumalatedSize` will not return 0

### 3.4. Conclusion

- From [3.1](#31-When-does-a-UE-gets-deleted-from-or-reordered-in-ueToBeScheduled-list), [3.2](#32-When-will-isDlMsgScheduled-be-TRUE), [3.3](#33-When-will-accumalatedSize--0), we can conclude that if there are remaining data for UE1 LC 3-2 in "[Prediction Example Case](#2-Prediction)", UE1 LC 3-2 allocated move to the back of UE 2 LC 3-2

## 4. Extra Code Trace: Where does UE get added into `ueToBeScheduled`?

- UE get added into `ueToBeScheduled` in `schFcfsAddUeToSchedule` function
```c=464
uint8_t schFcfsAddUeToSchedule(SchCellCb *cellCb, uint16_t ueIdToAdd)
{
    ...
```
```c=490
    ...
    if(addNodeToLList(&fcfsCellCb->ueToBeScheduled, ueId, NULLP) != ROK)
    {
        DU_LOG("\nERROR  --> SCH : Failed to add ueId [%d] to cell->ueToBeScheduled list", *ueId);
        return RFAILED;
    }
    return ROK;
}
```

- Who calls `schFcfsAddUeToSchedule`?
    - `schFcfsAddToDlHqRetxList`
    - `schFcfsAddToUlHqRetxList`
    - `schFcfsProcessCrcInd`
    - `schFcfsDlRlcBoInfo` (RLC BO Status indication)
    - `schFcfsBsr`
    - `schFcfsSrUciInd`
    - `schFcfsProcessRachInd`


- Who else calls `schFcfsAddUeToSchedule`?
    - `schMsg4FeedbackUpdate` through `SchAddUeToSchedule`
    - `schHdlDlHqRetxStrtTimerForDl` through `SchAddUeToSchedule`
    - `schHdlUlHqRetxStrtTimerForDl` through `SchAddUeToSchedule`


## 5. Example RUN

```shell=8654
...
DEBUG  -->  EGTP : Received DL Message[1]

DEBUG  -->  DU_APP : Processing DL data in duHdlEgtpDlData()
DEBUG  -->  DU_APP : Sending User Data Msg to RLC [TEID, nPDU]:[1, 1]

DEBUG  -->  RLC_DL : Received DL Data
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
INFO   --> SCH: SFN:350/Slot:7, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:2, AggLvl:1 and SymbolDuration2 with StartPrb:60, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:2
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[747,496,60]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:747,Idx:0, TotalBO Size:496
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:43 is less than reqPrb:59
DEBUG  -->  MAC: Send scheduled result report for sfn 350 slot 7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:496, lcId:4, total :496
DEBUG  -->  MAC: Received DL data for sfn=350 slot=7 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [749]
INFO   --> SCH: SFN:350/Slot:8, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:1, AggLvl:1 and SymbolDuration2 with StartPrb:57, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:1
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: SharedPRB is less
DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[288,464,57]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:288,Idx:0, TotalBO Size:464
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:46 is less than reqPrb:55
DEBUG  -->  MAC: Send scheduled result report for sfn 350 slot 8[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=350 slot=7
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:464, lcId:4, total :960
DEBUG  -->  MAC: Received DL data for sfn=350 slot=8 numPdu= 1
DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [292]
INFO   --> SCH: SFN:350/Slot:9, is a Valid PDCCH slot
DEBUG  -->  SCH: RBG found for cceIndex:5, AggLvl:1 and SymbolDuration2 with StartPrb:69, numPrb:3
INFO   -->  SCH: PDCCH allocation is successful at cceIndex:5
DEBUG  -->  SCH: LC:4 is the First node to be allocated which includes TX_PAYLOAD_HDR_LEN
DEBUG  -->  SCH: All LC are allocated [SharedPRB:31]
INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[0,295,38]
INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:0,Idx:0, TotalBO Size:295
DEBUG  -->  SCH: LCID4 Deleted successfully
INFO   --> SCH: In isPrbAvailable, numFreePrb:34 is less than reqPrb:37
DEBUG  -->  MAC: Send scheduled result report for sfn 350 slot 9[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=350 slot=8
DEBUG  -->  RLC : Received scheduling report from MAC
DEBUG  -->  RLC : SNSSAI found in LL
INFO   -->  RLC_DL: SNSSAI List Grant:295, lcId:4, total :1255
DEBUG  -->  MAC: Received DL data for sfn=350 slot=9 numPdu= 1[1;32m
DEBUG  -->  LWR_MAC: DL MSG sent...[0m
DEBUG  -->  LWR_MAC: Sending TX DATA Request
INFO   -->  PHY_STUB: PDCCH PDU
INFO   -->  PHY_STUB: PDSCH PDU
INFO   -->  PHY STUB: TX DATA Request at sfn=350 slot=9
...
```

- Explanation:
    1. ODU get DL message from OCU
    ```shell=8655
    DEBUG  -->  EGTP : Received DL Message[1]
    ```
    2. Total BO is 1240 for LCID 4. SCH received BO Status Indication from RLC and add the UE into ueToBeScheduled
    ```shell=8661
    DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [1240]
    ```
    3. SCH allocated PRB to DL message, however the available PRB is not enough
    ```shell=8666
    DEBUG  -->  SCH: SharedPRB is less
    DEBUG  -->  SCH: All LC are allocated [SharedPRB:0]
    INFO   -->  SCH : LcID:4, [reqBO, allocBO, allocPRB]:[747,496,60]
    INFO   -->  SCH: Added in MAC BO report: LCID:4,reqBO:747,Idx:0, TotalBO Size:496
    DEBUG  -->  SCH: LCID4 Deleted successfully
    INFO   --> SCH: In isPrbAvailable, numFreePrb:43 is less than reqPrb:59
    ```
    4. After this, UE has been deleted from ueToBeScheduled eventhough there are remaining BO to be scheduled in the next slot
    5. MAC and PHY_STUB will received the DL data
    ```shell=8676
    DEBUG  -->  MAC: Received DL data for sfn=350 slot=7 numPdu= 1
    ```
    ```shell=8693
    INFO   -->  PHY STUB: TX DATA Request at sfn=350 slot=7
    ```
    6. SCH received BO Status Indication from RLC and add the UE into ueToBeScheduled. Total BO is 749 for LCID 4 (reduced since some BO already been sent.
    ```shell=8677
    DEBUG  -->  SCH : Received RLC BO Status indication LCId [4] BO [749]
    ```
    7. Process repeats until all [1240] BO sent
