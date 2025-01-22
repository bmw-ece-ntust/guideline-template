# 2024/10/23 Study Note (ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN)

###### tags: `2024`


**Goal:**
- [x] [ORANSlice: An Open-Source 5G Network Slicing Platform for
O-RAN](#1-Paper-Summary)

**References:**
- [ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN](https://arxiv.org/abs/2410.12978)

**Table of Contents:**
- [2024/10/23 Study Note (ORANSlice: An Open-Source 5G Network Slicing Platform for O-RAN)](#2024-10-23-study-note--oranslice--an-open-source-5g-network-slicing-platform-for-o-ran-)
          + [tags: `2024`](#tags---2024-)
  * [1. Paper Summary](#1-paper-summary)
  * [2. What they add in OAI nrUE?](#2-what-they-add-in-oai-nrue-)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 1. Paper Summary

- Basically:
    -  The writer add slice aware MAC Scheduler in OAI gNB. Supported RRM Policy:
        -  Min Ratio
        -  Max Ratio
    -  The writer add multiple PDU session (also slice aware) for OAI nrUE
    -  The writer also add E2SM-CCC for slicing (rrm policy) only

- Description of the Test Architecture:
![image](https://hackmd.io/_uploads/r1PDBz8l1g.png)

- Test Slicing xApp can send Max Ratio to gNB:
![image](https://hackmd.io/_uploads/rkei6rfUgJx.png)

- Test Slicing xApp can send Min Ratio to gNB
![image](https://hackmd.io/_uploads/r1El8M8gkg.png)

- Code: 
    - NEU Lab's Github =
        - OAI-Slicing-Intel (Development version) https://github.com/wineslab/OAI-Slicing-Intel
        - ORANSlice (Final version) https://github.com/wineslab/ORANSlice
    - OAI's Gitlab = https://gitlab.eurecom.fr/oai/openairinterface5g/-/tree/NR_UE_multi_pdu?ref_type=heads

## 2. What they add in OAI nrUE?

- First, they create a new executable, called `ue_slice_manager`. This executable is the one who can read user input for making new PDU Session with specific slice
![image](https://hackmd.io/_uploads/S1HdqQLlke.png)

- Example code snippet that prove this executable can make PDU Session with specific slice ([`ue_slice_manager.c`](https://github.com/wineslab/OAI-Slicing-Intel/blob/NR_UE_multi_pdusession/openair2/SLICING/ue_slice_manager.c#L68C1-L119C2)):
```c=68
static void get_user_command(uint8_t cmd_input[8], sdap_entity_info_t info)
{
  printf("Enter any of the following commands to send\n");
  printf("--Request new PDU session: 1\n");
  printf("--Request deletion of PDU session: 2\n");
  printf("--Request active PDU session info: 3\n");
  printf("\n");

  int cmd;
  int pdu_id;
  int snssai_idx;
  bool pdu_id_exists = false;
  printf("Enter the option here: ");
  scanf("%d", &cmd);

  if (cmd == 1) {
    printf("Enter new PDU session id: ");
    scanf("%d", &pdu_id);
    pdu_id_exists = check_pdu_exists(pdu_id, info);
    if (pdu_id_exists) {
      printf("Entered PDU is already active\n");
      get_user_command(cmd_input, info);
    } else {
      cmd_input[0] = cmd;
      cmd_input[1] = pdu_id;
    }
    printf("Enter S-NSSAI index from allowed NSSAI list: ");
    scanf("%d", &snssai_idx);
    if (snssai_idx < 7 && snssai_idx >= 0) {
      cmd_input[2] = snssai_idx;
    } else {
      printf("Entered S-NSSAI index out of bound. Should be from 0 to 7.\n");
      get_user_command(cmd_input, info);
    }
  } else if (cmd == 2) {
    printf("Enter PDU session id to delete: ");
    scanf("%d", &pdu_id);
    pdu_id_exists = check_pdu_exists(pdu_id, info);
    if (pdu_id_exists) {
      cmd_input[0] = cmd;
      cmd_input[1] = pdu_id;
    } else {
      printf("Entered PDU session not active\n");
      get_user_command(cmd_input, info);
    } 
  } else if (cmd == 3) {
    cmd_input[0] = cmd;
  } else {
    printf("Invalid option\n");
    get_user_command(cmd_input, info);
  }
}
```

- Then, they add some modification to the OAI nrUE. Mainly, they add a receiver for `ue_slice_manager` requests by creating a new thread called [`nas_user_mngr`](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/NR_UE_multi_pdu/openair3/NAS/NR_UE/nr_nas_msg_sim.c#L1665):
```c=1665
static void *nas_user_mngr(void *args)
{
  const int ue_id = 0; // create user manager for only one UE instance
  user_api_id_t user_api = {0};
  at_response_t atr = {0};
  user_at_commands_t atc = {0};

  if (user_api_initialize(&user_api, NULL, NULL, get_nrUE_params()->at_interface_name, "9600") != RETURNok) {
    LOG_E(NAS, "Unable to initialize NAS user interface\n");
  }
  nr_ue_nas_t *nas = get_ue_nas_info(ue_id);
  init_nas_user(nas);
  nas_user_t *nas_user = nas->nas_user;
  nas_user->ueid = nas->UE_id;
  nas_user->nas_ue_type = UE_NR;
  nas_user->nas_reset_pdn = nas_reset_pdp_context;
  nas_user->nas_get_pdn = nas_get_pdp_contexts;
  nas_user->nas_set_pdn = nas_set_pdp_context;
  nas_user->nas_get_pdn_range = nas_get_pdp_range;
  nas_user->nas_deactivate_pdn = nas_request_pdu_release;
  nas_user->nas_activate_pdn = nas_request_pdusession;
  nas_user->nas_get_pdn_status = nas_get_pdp_status;
  nas_user->user_api_id = &user_api;
  nas_user->at_response = &atr;
  nas_user->user_at_commands = &atc;

  nas_user_context_t user_ctxt = {0};
  const char *version = "v1.0";
  nas_user_context_initialize(&user_ctxt, version);
  user_ctxt.sim_status = NAS_USER_READY; /* set SIM ready by default */
  nas_user->nas_user_context = &user_ctxt;

  bool exit_loop = false;

  int fd = user_api_get_fd(&user_api);
  LOG_I(NAS, "User interface manager started with fd %d\n", fd);

  while (!exit_loop) {
    exit_loop = nas_user_receive_and_process(nas_user, NULL);
  }

  user_api_close(nas_user->user_api_id);
  LOG_I(NAS, "User interface manager ended\n");
  free_and_zero(nas->nas_user);

  return NULL;
}
```

- Upon receiving PDU Session Establishment Command, this thread will send msg to `nas_nrue_task`
```plantuml
participant "nas_user_mngr" as mngr
participant "nas_nrue_task" as nas

note over mngr:nas_user_receive_and_process()
note over mngr:nas_user_process_data()
note over mngr:_nas_user_procedure()\n = _nas_user_proc_cgact()
note over mngr:nas_activate_pdn()
note over mngr:nas_request_pdusession()
note over mngr:request_pdusession()
mngr-->nas:itti_send_msg_to_task(TASK_NAS_NRUE)
note over nas:generatePduSessionEstablishRequest()
note over nas:send_nas_uplink_data_req()
```

- Note, when Running OAI nrUE, flag `--net-slice` should be set
![image](https://hackmd.io/_uploads/SyNnbEIx1x.png)
