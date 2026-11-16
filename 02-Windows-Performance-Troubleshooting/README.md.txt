                                          Project 2 — Windows Performance Troubleshooting


# Project 2 — Windows Performance Troubleshooting

## 1. Issue / Problem

The Windows 10 system was experiencing intermittent periods of high CPU utilization. At certain points during normal use, CPU usage would increase significantly before eventually returning to a lower level.

The issue was not constant, so the main challenge was to determine what was causing the periods of elevated CPU activity and whether a specific process was responsible.

The objective of this troubleshooting task was to use Windows built-in diagnostic tools to monitor CPU activity, identify the process or system activity associated with the higher usage, investigate the behavior, and verify the system's performance after corrective action was taken.

---

## 2. Symptoms / Initial Observations

The first symptom observed was an increase in overall CPU utilization in Windows Task Manager. The CPU percentage would rise noticeably during periods of activity and later fall back to a lower level.

Because the CPU usage was fluctuating rather than remaining continuously high, Task Manager was used to monitor the system over time and identify which processes were consuming the most CPU when the spikes occurred.

During the initial investigation, the system showed a significant CPU spike. By monitoring the individual processes in Task Manager, it became possible to narrow the investigation to the processes contributing to the increased CPU usage.

The system was monitored before, during, and after the periods of elevated CPU activity so that the behavior could be compared and documented accurately.

---

## 3. Investigation & Troubleshooting

### 3.1 Monitoring CPU Activity

Windows Task Manager was used as the primary troubleshooting tool because it provides a real-time view of overall CPU utilization as well as the CPU usage of individual processes.

The CPU column was monitored during the periods of increased activity to determine which process was responsible for the higher utilization.

**Evidence:** Initial Task Manager observation showing the system's CPU activity before and during the investigation.

### 3.2 Identifying the High-CPU Process

During one of the sustained CPU events, PowerShell became the most significant CPU-consuming process observed in Task Manager.

PowerShell was recorded using approximately **51.6% CPU** during the high-usage condition. This made it the primary process requiring further investigation.

The process was monitored to determine whether the elevated CPU usage continued and whether it was directly related to the observed system performance issue.

**Evidence:** Task Manager screenshot showing PowerShell consuming approximately 51.6% CPU.

### 3.3 Taking Corrective Action

After confirming that the PowerShell activity was responsible for the sustained CPU usage observed during the event, the PowerShell activity was stopped.

Following the action, CPU utilization decreased significantly. A subsequent Task Manager observation showed CPU usage at approximately **3%**.

This provided an initial indication that the sustained high-CPU condition had been resolved.

**Evidence:** Task Manager screenshot showing the system after the PowerShell activity had stopped and CPU utilization had returned to a low level.

### 3.4 Continued Monitoring

The system was not considered fully resolved immediately after the CPU dropped. Task Manager was kept open and the system was monitored further to determine whether the high CPU condition returned.

During this continued observation, additional CPU spikes occurred. However, these later spikes were brief and returned to a low level without remaining continuously elevated.

Service Host processes (`svchost.exe`) were observed among the processes contributing to some of these temporary spikes.

### 3.5 Investigating Service Host

Because Service Host appeared during the later CPU spikes, the relevant Service Host group was expanded in Task Manager to identify the Windows services running within it.

The investigation identified **PID 1032** as the relevant Service Host process. Further inspection showed that multiple Windows services were associated with the same PID, including:

- SysMain
- Windows Audio Endpoint Builder
- Other Windows services within the same Service Host group

This was an important finding because `svchost.exe` is a host process that can run multiple Windows services under the same process ID.

Therefore, the CPU usage observed from the shared Service Host process could not be attributed to SysMain or another individual service solely from the shared PID.

**Evidence:** Task Manager screenshot showing the expanded Service Host group and the Windows services operating within it.

### 3.6 Final Observation

The system continued to be monitored after the PowerShell activity had been stopped.

CPU utilization generally remained within a low range of approximately **3–7%**, although short CPU spikes continued to occur periodically.

Unlike the original PowerShell event, these later spikes did not remain consistently elevated. They rose and returned to lower levels during normal system activity.

Based on the observations collected during the investigation, the sustained CPU condition was associated with the PowerShell activity. The later Service Host activity was investigated but was not identified as the root cause of the original sustained CPU condition.

---

## 4. Troubleshooting Outcome

The investigation successfully identified and addressed the sustained CPU condition observed during the troubleshooting session.

The main findings were:

- Task Manager was used to establish and monitor CPU activity.
- PowerShell was identified consuming approximately **51.6% CPU** during the sustained high-usage event.
- The PowerShell activity was stopped.
- CPU utilization subsequently decreased to approximately **3–7%**.
- Additional intermittent CPU spikes were observed during continued monitoring.
- Service Host was investigated because it appeared during some of the later spikes.
- Multiple Windows services were found operating under the same Service Host process/PID.
- No individual Service Host service was identified as the cause of the original sustained CPU condition based on the available evidence.

The troubleshooting demonstrated the importance of identifying the specific process associated with a performance issue before taking corrective action, as well as continuing to monitor the system after the corrective action to verify the result.