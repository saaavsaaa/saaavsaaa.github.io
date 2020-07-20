slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1233
slurmd: debug3: task/affinity: job 1233 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1233 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1233 CPU final HW mask for node: 0x03
slurmd: debug3: state for jobid 24: ctime:1589768149 revoked:0 expires:0
slurmd: debug3: state for jobid 25: ctime:1589770622 revoked:0 expires:0
slurmd: debug3: state for jobid 52: ctime:1589959961 revoked:0 expires:0
slurmd: debug3: state for jobid 1206: ctime:1595233974 revoked:0 expires:0
slurmd: debug3: state for jobid 1207: ctime:1595234075 revoked:0 expires:0
slurmd: debug3: state for jobid 1208: ctime:1595234088 revoked:0 expires:0
slurmd: debug3: state for jobid 1209: ctime:1595234251 revoked:1595234251 expires:1595234251
slurmd: debug3: destroying job 1209 state
slurmd: debug3: state for jobid 1209: ctime:1595234251 revoked:0 expires:0
slurmd: debug3: state for jobid 1212: ctime:1595234313 revoked:1595234313 expires:1595234313
slurmd: debug3: destroying job 1212 state
slurmd: debug3: state for jobid 1210: ctime:1595234313 revoked:1595234313 expires:1595234313
slurmd: debug3: destroying job 1210 state
slurmd: debug3: state for jobid 1214: ctime:1595234313 revoked:1595234313 expires:1595234313
slurmd: debug3: destroying job 1214 state
slurmd: debug3: state for jobid 1215: ctime:1595234313 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1215 state
slurmd: debug3: state for jobid 1213: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1213 state
slurmd: debug3: state for jobid 1218: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1218 state
slurmd: debug3: state for jobid 1219: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1219 state
slurmd: debug3: state for jobid 1216: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1216 state
slurmd: debug3: state for jobid 1220: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1220 state
slurmd: debug3: state for jobid 1222: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1222 state
slurmd: debug3: state for jobid 1217: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1217 state
slurmd: debug3: state for jobid 1223: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1223 state
slurmd: debug3: state for jobid 1224: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1224 state
slurmd: debug3: state for jobid 1221: ctime:1595234314 revoked:1595234314 expires:1595234314
slurmd: debug3: destroying job 1221 state
slurmd: debug3: state for jobid 1226: ctime:1595234317 revoked:1595234317 expires:1595234317
slurmd: debug3: destroying job 1226 state
slurmd: debug3: state for jobid 1227: ctime:1595234317 revoked:1595234317 expires:1595234317
slurmd: debug3: destroying job 1227 state
slurmd: debug3: state for jobid 1225: ctime:1595234317 revoked:1595234317 expires:1595234317
slurmd: debug3: destroying job 1225 state
slurmd: _run_prolog: run job script took usec=7
slurmd: _run_prolog: prolog with lock for job 1233 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1233 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1233
slurmd: debug:  credential for job 1233 revoked
slurmd: debug2: No steps in jobid 1233 to send signal 999
slurmd: debug2: No steps in jobid 1233 to send signal 18
slurmd: debug2: No steps in jobid 1233 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1233 to 1595235152 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1228
slurmd: debug3: task/affinity: job 1228 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1228 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1228 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=6
slurmd: _run_prolog: prolog with lock for job 1228 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1228 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1228
slurmd: debug:  credential for job 1228 revoked
slurmd: debug2: No steps in jobid 1228 to send signal 999
slurmd: debug2: No steps in jobid 1228 to send signal 18
slurmd: debug2: No steps in jobid 1228 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1228 to 1595235152 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1234
slurmd: debug3: task/affinity: job 1234 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1234 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1234 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=6
slurmd: _run_prolog: prolog with lock for job 1234 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1234 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1234
slurmd: debug:  credential for job 1234 revoked
slurmd: debug2: No steps in jobid 1234 to send signal 999
slurmd: debug2: No steps in jobid 1234 to send signal 18
slurmd: debug2: No steps in jobid 1234 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1234 to 1595235152 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1235
slurmd: debug3: task/affinity: job 1235 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1235 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1235 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=6
slurmd: _run_prolog: prolog with lock for job 1235 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1235 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1235
slurmd: debug:  credential for job 1235 revoked
slurmd: debug2: No steps in jobid 1235 to send signal 999
slurmd: debug2: No steps in jobid 1235 to send signal 18
slurmd: debug2: No steps in jobid 1235 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1235 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1229
slurmd: debug3: task/affinity: job 1229 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1229 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1229 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=5
slurmd: _run_prolog: prolog with lock for job 1229 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1229 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1229
slurmd: debug:  credential for job 1229 revoked
slurmd: debug2: No steps in jobid 1229 to send signal 999
slurmd: debug2: No steps in jobid 1229 to send signal 18
slurmd: debug2: No steps in jobid 1229 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1229 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1236
slurmd: debug3: task/affinity: job 1236 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1236 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1236 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=5
slurmd: _run_prolog: prolog with lock for job 1236 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1236 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1236
slurmd: debug:  credential for job 1236 revoked
slurmd: debug2: No steps in jobid 1236 to send signal 999
slurmd: debug2: No steps in jobid 1236 to send signal 18
slurmd: debug2: No steps in jobid 1236 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1236 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1237
slurmd: debug3: task/affinity: job 1237 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1237 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1237 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=5
slurmd: _run_prolog: prolog with lock for job 1237 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1237 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1237
slurmd: debug:  credential for job 1237 revoked
slurmd: debug2: No steps in jobid 1237 to send signal 999
slurmd: debug2: No steps in jobid 1237 to send signal 18
slurmd: debug2: No steps in jobid 1237 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1237 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1230
slurmd: debug3: task/affinity: job 1230 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1230 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1230 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=5
slurmd: _run_prolog: prolog with lock for job 1230 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1230 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1230
slurmd: debug:  credential for job 1230 revoked
slurmd: debug2: No steps in jobid 1230 to send signal 999
slurmd: debug2: No steps in jobid 1230 to send signal 18
slurmd: debug2: No steps in jobid 1230 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1230 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1238
slurmd: debug3: task/affinity: job 1238 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1238 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1238 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=6
slurmd: _run_prolog: prolog with lock for job 1238 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1238 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1238
slurmd: debug:  credential for job 1238 revoked
slurmd: debug2: No steps in jobid 1238 to send signal 999
slurmd: debug2: No steps in jobid 1238 to send signal 18
slurmd: debug2: No steps in jobid 1238 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1238 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1239
slurmd: debug3: task/affinity: job 1239 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1239 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1239 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=10
slurmd: _run_prolog: prolog with lock for job 1239 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1239 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1239
slurmd: debug:  credential for job 1239 revoked
slurmd: debug2: No steps in jobid 1239 to send signal 999
slurmd: debug2: No steps in jobid 1239 to send signal 18
slurmd: debug2: No steps in jobid 1239 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1239 to 1595235153 UTS
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 4005
slurmd: debug2: Processing RPC: REQUEST_BATCH_JOB_LAUNCH
slurmd: task_p_slurmd_batch_request: 1231
slurmd: debug3: task/affinity: job 1231 core mask from slurmctld: 0x3
slurmd: task/affinity: job 1231 CPU input mask for node: 0x03
slurmd: debug3: _lllp_map_abstract_masks
slurmd: task/affinity: job 1231 CPU final HW mask for node: 0x03
slurmd: _run_prolog: run job script took usec=7
slurmd: _run_prolog: prolog with lock for job 1231 ran for 0 seconds
slurmd: get env for user manager here
slurmd: Launching batch job 1231 for UID 1000
slurmd: debug3: _rpc_batch_job: call to _forkexec_slurmstepd
slurmd: debug3: slurmstepd rank -1 (sl311a-t-speech3), parent rank -1 (NONE), children 0, depth 0, max_depth 0
slurmd: debug3: _send_slurmstepd_init: call to getpwuid_r
slurmd: debug3: _send_slurmstepd_init: return from getpwuid_r
slurmd: debug3: _rpc_batch_job: return from _forkexec_slurmstepd: 0
slurmd: debug3: in the service_connection
slurmd: debug2: got this type of message 6011
slurmd: debug2: Processing RPC: REQUEST_TERMINATE_JOB
slurmd: debug:  _rpc_terminate_job, uid = 1000
slurmd: debug:  task_p_slurmd_release_resources: affinity jobid 1231
slurmd: debug:  credential for job 1231 revoked
slurmd: debug2: No steps in jobid 1231 to send signal 999
slurmd: debug2: No steps in jobid 1231 to send signal 18
slurmd: debug2: No steps in jobid 1231 to send signal 15
slurmd: debug4: sent ALREADY_COMPLETE
slurmd: debug2: set revoke expiration for jobid 1231 to 1595235153 UTS






记录使用Kaldi过程中遇到的错误：
  
kaldi/egs/multi_cn ：     


-----

    tail: cannot open 'exp/make_mfcc/primewords/train/make_mfcc_pitch_train.1.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 68 seems to no longer exists:
    'squeue -j 68' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/primewords/train/q/done.5145.1 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/primewords/train/make_mfcc_pitch_train.1.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified
    tail: cannot open 'exp/make_mfcc/stcmds/train/make_mfcc_pitch_train.1.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 69 seems to no longer exists:
    'squeue -j 69' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/stcmds/train/q/done.5215.1 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/stcmds/train/make_mfcc_pitch_train.1.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified
    tail: cannot open 'exp/make_mfcc/aishell/train/make_mfcc_pitch_train.1.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 70 seems to no longer exists:
    'squeue -j 70' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/aishell/train/q/done.5240.1 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/aishell/train/make_mfcc_pitch_train.1.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified
    tail: cannot open 'exp/make_mfcc/aidatatang/train/make_mfcc_pitch_train.1.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 77 seems to no longer exists:
    'squeue -j 77' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/aidatatang/train/q/done.5273.1 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/aidatatang/train/make_mfcc_pitch_train.1.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified
    tail: cannot open 'exp/make_mfcc/magicdata/train/make_mfcc_pitch_train.1.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 80 seems to no longer exists:
    'squeue -j 80' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/magicdata/train/q/done.5329.1 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/magicdata/train/make_mfcc_pitch_train.1.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified
    tail: cannot open 'exp/make_mfcc/thchs/train/make_mfcc_pitch_train.2.log' for reading: No such file or directory
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: Error: Job 65 seems to no longer exists:
    'squeue -j 65' returned error code 1 and said:
      slurm_load_jobs error: Invalid job id specified

    Syncfile exp/make_mfcc/thchs/train/q/done.5061.2 does not exist, meaning that the job did not finish.
    Log is in exp/make_mfcc/thchs/train/make_mfcc_pitch_train.2.log. Last line '' does not end in 'status 0'.
    Possible reasons:
      a) Exceeded time limit? -> Use more jobs!
      b) Shutdown/Frozen machine? -> Run again! squeue:
    slurm_load_jobs error: Invalid job id specified

-----
slurmd: error: SlurmdUser must be root to use --get-user-env      
slurmd: error: Unable to get user's local environment, running only with passed environment     
slurm/src/common/env.c :     
```
char **env_array_user_default(const char *username, int timeout, int mode,bool no_cache)
{     
　　...     
　　if (geteuid() != (uid_t)0) {     
　　error("SlurmdUser must be root to use --get-user-env");     
　　return NULL;     
}
```    









###### 服务器所在网络问题：

-----

    --- Preparing pronunciations for OOV words ...
    Traceback (most recent call last):
      File "/export1/kaldi/tools/sequitur-g2p/bin/g2p.py", line 4, in <module>
        __import__('pkg_resources').run_script('sequitur-g2p==1.0.1668.6', 'g2p.py')
      File "/home/manager/anaconda3/lib/python3.7/site-packages/pkg_resources/__init__.py", line 667, in run_script
        self.require(requires)[0].run_script(script_name, ns)
      File "/home/manager/anaconda3/lib/python3.7/site-packages/pkg_resources/__init__.py", line 1463, in run_script
        exec(code, namespace, namespace)
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/EGG-INFO/scripts/g2p.py", line 304, in <module>
        tool.run(main, options, args)
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/tool.py", line 63, in run
        status = runMain(main, options, args)
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/tool.py", line 99, in runMain
        status = main(options, args)
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/EGG-INFO/scripts/g2p.py", line 230, in main
        model = SequiturTool.procureModel(options, loadSample, log=log_stdout)
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/SequiturTool.py", line 217, in procureModel
        return tool.procureModel()
      File "/export1/kaldi/tools/sequitur-g2p/lib/python3.7/site-packages/sequitur_g2p-1.0.1668.6-py3.7-linux-x86_64.egg/SequiturTool.py", line 163, in procureModel
        model = pickle.load(open(self.options.modelFile, 'rb'), encoding='latin1')
    EOFError: Ran out of input
-----

查了下 cat local/prepare_dict.sh，由于服务器环境原因 wget http://sourceforge.net/projects/kaldi/files/sequitur-model4 -O conf/g2p_model 下载不下来，但是conf/g2p_model文件存在了，本地下载了上传上去就可以了     

###### 集群批处理失败：

-----

    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: invalid partition specified: shared
    sbatch: error: Batch job submission failed: Invalid partition name specified
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  -p shared --mem-per-cpu 2G  --open-mode=append -e exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: invalid partition specified: shared
    sbatch: error: Batch job submission failed: Invalid partition name specified
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: invalid partition specified: shared
    sbatch: error: Batch job submission failed: Invalid partition name specified
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p shared  --open-mode=append -e exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: invalid partition specified: shared
    sbatch: error: Batch job submission failed: Invalid partition name specified
    steps/train_mono.sh --boost-silence 1.25 --nj 20 --cmd slurm.pl --mem 2G data/aishell/train data/lang exp/mono
    steps/train_mono.sh: Initializing monophone system.
    feat-to-dim 'ark,s,cs:apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- | add-deltas ark:- ark:- |' - 
    apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- 
    WARNING (apply-cmvn[5.5.683~1-6ef3f]:Open():util/kaldi-table-inl.h:106) Failed to open script file data/aishell/train/split20/1/feats.scp
    add-deltas ark:- ark:- 
    ERROR (apply-cmvn[5.5.683~1-6ef3f]:SequentialTableReader():util/kaldi-table-inl.h:860) Error constructing TableReader: rspecifier is scp:data/aishell/train/split20/1/feats.scp

    [ Stack-Trace: ]
    /export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7fe2cb6f4682]
    apply-cmvn(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x5623208b2f2f]
    apply-cmvn(kaldi::SequentialTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::SequentialTableReader(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)+0xc2) [0x5623208ba540]
    apply-cmvn(main+0x79b) [0x5623208b0985]
    /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7fe2cab5eb97]
    apply-cmvn(_start+0x2a) [0x5623208b010a]

    kaldi::KaldiFatalErrorERROR (feat-to-dim[5.5.683~1-6ef3f]:main():feat-to-dim.cc:58) Could not read any features (empty archive?)

    [ Stack-Trace: ]
    /export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7f519921b682]
    feat-to-dim(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x558ad959e3f1]
    feat-to-dim(main+0x2e9) [0x558ad959d7e3]
    /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7f5198685b97]
    feat-to-dim(_start+0x2a) [0x558ad959d41a]

    kaldi::KaldiFatalErrorerror getting feature dimension

-----

由于默认的sbatch -p 是shared，和我的不一样，改下参数就好了。

###### 内存资源不足及配置依赖:

------

    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/thchs/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/thchs/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/thchs/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/thchs/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/thchs/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    utils/validate_data_dir.sh: Successfully validated data-directory data/primewords/train
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/primewords/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/primewords/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/primewords/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/primewords/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/primewords/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    utils/validate_data_dir.sh: Successfully validated data-directory data/stcmds/train
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    utils/validate_data_dir.sh: Successfully validated data-directory data/aishell/train
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/stcmds/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/stcmds/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/stcmds/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/stcmds/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/stcmds/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aishell/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/aishell/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/aishell/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aishell/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/aishell/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    utils/validate_data_dir.sh: Successfully validated data-directory data/aidatatang/train
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aidatatang/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/aidatatang/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/aidatatang/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aidatatang/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/aidatatang/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    utils/validate_data_dir.sh: Successfully validated data-directory data/magicdata/train
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/magicdata/train/q/make_mfcc_pitch_train.log, command was sbatch --export=PATH  --ntasks-per-node=1  -p compute --mem-per-cpu 2G  --open-mode=append -e exp/make_mfcc/magicdata/train/q/make_mfcc_pitch_train.log -o exp/make_mfcc/magicdata/train/q/make_mfcc_pitch_train.log --array 1-20 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/magicdata/train/q/make_mfcc_pitch_train.sh >>exp/make_mfcc/magicdata/train/q/make_mfcc_pitch_train.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    steps/make_mfcc_pitch_online.sh --cmd slurm.pl --mem 2G --nj 10 data/aishell/test exp/make_mfcc/aishell/test mfcc/aishell
    steps/make_mfcc_pitch_online.sh --cmd slurm.pl --mem 2G --nj 10 data/aidatatang/test exp/make_mfcc/aidatatang/test mfcc/aidatatang
    steps/make_mfcc_pitch_online.sh --cmd slurm.pl --mem 2G --nj 10 data/magicdata/test exp/make_mfcc/magicdata/test mfcc/magicdata
    steps/make_mfcc_pitch_online.sh --cmd slurm.pl --mem 2G --nj 10 data/thchs/test exp/make_mfcc/thchs/test mfcc/thchs
    utils/validate_data_dir.sh: Successfully validated data-directory data/thchs/test
    utils/validate_data_dir.sh: Successfully validated data-directory data/aishell/test
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/thchs/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    utils/validate_data_dir.sh: Successfully validated data-directory data/magicdata/test
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  --mem-per-cpu 2G -p compute  --open-mode=append -e exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aishell/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  -p compute --mem-per-cpu 2G  --open-mode=append -e exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/magicdata/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    utils/validate_data_dir.sh: Successfully validated data-directory data/aidatatang/test
    steps/make_mfcc_pitch_online.sh: [info]: no segments file exists: assuming wav.scp indexed by utterance.
    /export1/kaldi/egs/multi_cn/s5/utils/slurm.pl: error submitting jobs to queue (return status was 256)
    queue log file is exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log, command was sbatch --export=PATH  --ntasks-per-node=1  -p compute --mem-per-cpu 2G  --open-mode=append -e exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log -o exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log --array 1-10 /export1/kaldi/egs/multi_cn/s5/exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.sh >>exp/make_mfcc/aidatatang/test/q/make_mfcc_pitch_test.log 2>&1
    sbatch: error: Memory specification can not be satisfied
    sbatch: error: Batch job submission failed: Requested node configuration is not available
    steps/train_mono.sh --boost-silence 1.25 --nj 20 --cmd slurm.pl --mem 2G data/aishell/train data/lang exp/mono
    steps/train_mono.sh: Initializing monophone system.
    feat-to-dim 'ark,s,cs:apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- | add-deltas ark:- ark:- |' - 
    apply-cmvn --utt2spk=ark:data/aishell/train/split20/1/utt2spk scp:data/aishell/train/split20/1/cmvn.scp scp:data/aishell/train/split20/1/feats.scp ark:- 
    WARNING (apply-cmvn[5.5.683~1-6ef3f]:Open():util/kaldi-table-inl.h:106) Failed to open script file data/aishell/train/split20/1/feats.scp
    ERROR (apply-cmvn[5.5.683~1-6ef3f]:SequentialTableReader():util/kaldi-table-inl.h:860) Error constructing TableReader: rspecifier is scp:data/aishell/train/split20/1/feats.scp

    [ Stack-Trace: ]
    /export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7fb16f784682]
    apply-cmvn(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x558930a26f2f]
    apply-cmvn(kaldi::SequentialTableReader<kaldi::KaldiObjectHolder<kaldi::Matrix<float> > >::SequentialTableReader(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)+0xc2) [0x558930a2e540]
    apply-cmvn(main+0x79b) [0x558930a24985]
    /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7fb16ebeeb97]
    apply-cmvn(_start+0x2a) [0x558930a2410a]

    kaldi::KaldiFatalErroradd-deltas ark:- ark:- 
    ERROR (feat-to-dim[5.5.683~1-6ef3f]:main():feat-to-dim.cc:58) Could not read any features (empty archive?)

    [ Stack-Trace: ]
    /export1/kaldi/src/lib/libkaldi-base.so(kaldi::MessageLogger::LogMessage() const+0xb42) [0x7f67d2457682]
    feat-to-dim(kaldi::MessageLogger::LogAndThrow::operator=(kaldi::MessageLogger const&)+0x21) [0x560a9e1763f1]
    feat-to-dim(main+0x2e9) [0x560a9e1757e3]
    /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7f67d18c1b97]
    feat-to-dim(_start+0x2a) [0x560a9e17541a]

    kaldi::KaldiFatalErrorerror getting feature dimension

-----

$scontrol show nodes发现有个节点，FreeMem=642不到1G了    
     srun -N2 --mem=2GB /bin/hostname    
     srun: error: Memory specification can not be satisfied    
     srun: Force Terminated job 36    
     srun: error: Unable to allocate resources: Requested node configuration is not available    
$free -g     
|           | total   |    used   |    free  |   shared | buff/cache | available  |
|:-:|--:|--:|--:|--:|--:|--:|
|Mem:       |    23   |       0   |       0  |        0 |        22  |       22   | 
|Swap:      |     7   |       0   |       7  |  |||

https://slurm.schedmd.com/sbatch.html   --mem 参数     
NOTE: Enforcement of memory limits currently relies upon the task/cgroup plugin or enabling of accounting     
 scontrol show config     
