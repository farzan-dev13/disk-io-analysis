## 🧠Investigating High Disk I/O in Linux - Using iostat, iotop, and Visual Analysis

## **Introduction: What is Disk I/O and Why Monitor It**

In Linux systems, disk performance is crucial for overall system responsiveness. A common reason behind system sluggishness is high disk I/O usage. This happens when the disk cannot keep up with the number of read/write operations being requested. It could be due to hardware limitations, application overload, or inefficient background processes.

To investigate these issues, Linux provides several tools. One of the most fundamental and useful is the `iostat` command.



## **Understanding the `iostat` Command Output**

The `iostat` tool reports CPU and device utilization statistics. When analyzing disk performance, the following columns are particularly important:

**1. `%util` – Device Utilization**

This column shows how much time the disk spends servicing I/O requests.

If `%util` is close to 100%, the disk is almost always busy, and it may indicate a performance bottleneck.


## Rule of Thumb:

- Below 30%: Normal

- 50%–70%: High usage

- Above 90%: Bottleneck likely

## Example:

```
Device: sda   %util: 92.65
```

This means the disk is busy 92.65% of the time — a strong indicator of saturation.


## 2. await – Average Wait Time (in milliseconds)

This represents how long I/O requests wait before being serviced. It includes both the time spent waiting in the queue and the time spent servicing the request.


## Expected Values:

- SSD: usually < 10 ms

- HDD: usually < 20 ms

```
Device: sda   await: 10.20 ms
```

This is acceptable for an SSD but could be slow for an HDD if it increases under load.

3. `r/s` and `w/s` – Read/Write Requests Per Second

These indicate how many read `r/s` and write `w/s` operations are being issued per second. They help identify the nature of the disk workload.

## Example:
```
Device: sda   r/s: 1.55   w/s: 44.48
```

Heavy write activity is observed here, which may point to log writing or database operations.

## Key Takeaways:
- **High `%util`**: Disk is heavily loaded — investigate what's using it.

- **High `await`**: Disk is slow in responding — check for overloaded processes or hardware limitations.

- Disproportionate **`r/s` or `w/s`**: Use tools like <code>iotop</code> or <code>dstat</code> to find the culprit processes.

In the next section, we will use `iotop` to pinpoint which processes are generating the most I/O and how to mitigate the impact.



pic



## 🔍 Step 2: Identify I/O-Heavy Processes with `iotop`

While `iostat` gives us a system-wide view of disk performance, it doesn't tell us which process is causing high I/O. That’s where `iotop` comes in — it’s like `top`, but for disk usage.

## 🔧 Installing `iotop`
If it's not already installed, you can get it with:
```
sudo apt install iotop
```

**Note: You need root privileges to run `iotop`.**

🚀 Basic Usage

Run it with:

```
sudo iotop
```

You’ll see a real-time list of processes with columns like:

| Column       | Meaning                                    |
| ------------ | ------------------------------------------ |
| `PID`        | Process ID                                 |
| `USER`       | Owner of the process                       |
| `DISK READ`  | Current read rate (e.g. KB/s or MB/s)      |
| `DISK WRITE` | Current write rate                         |
| `SWAPIN`     | % of process swapped in                    |
| `IO`         | % of time the process spent waiting on I/O |
| `COMMAND`    | The actual command or process name         |


## 🎯 What to Look For

- Look for processes with high IO percentage. This shows they are waiting on the disk.

- If a background process like tracker3 or snapd is constantly at the top, consider stopping or disabling it.



⚙️ High DISK WRITE or DISK READ values can signal data-intensive operations like:

- Browser caching

- Backup utilities

- Database indexing


✂️ Reducing the Impact

Here are some tips:

-Use nice or ionice to lower the priority of I/O-heavy processes:
```
sudo ionice -c3 -p <PID>
```

This sets the process to "idle" I/O priority.

- Kill or stop non-critical high-I/O processes:
```
sudo kill <PID>
```

- Consider excluding aggressive indexers like tracker3 if not needed:
```
gsettings set org.freedesktop.Tracker3.Miner.Files enable-monitors false
```

## ⚖️ Step 3: Visualizing Disk I/O – HDD vs SSD Behavior


Understanding how different types of disks behave under load is crucial when analyzing performance. Solid State Drives (SSDs) and Hard Disk Drives (HDDs) react very differently to high I/O situations.

In this section, we'll use a custom Python script to visualize disk usage, helping us identify slowdowns and saturation in real-time.


## **📊 Why Compare HDD and SSD Behavior?**
- HDDs have mechanical parts, so they suffer from seek time and rotational latency.

- SSDs use flash memory and offer much faster access times.

- However, both can become saturated under load, just in different ways.



## 🧪 Our Approach
We'll:
Use a Python script with `psutil` and `shutil` to show I/O load in real-time

Identify patterns (like heavy writes or frequent reads)

Compare how long operations take on SSD vs HDD


The following visual shows a snapshot of current mount points and their disk usage, captured using a custom monitoring tool.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pvcyr02e6tq7dvh5chqw.png)


**🗂️ Note:**
The reason many partitions appear with 100% usage here is that Snap applications are mounted as separate loop devices. They are read-only images and don't actually consume additional disk space beyond their original size.
You can usually ignore these entries when investigating real disk usage issues.


## **🗂️ Side Note:**

Mounted Snap Partitions and Disk Usage Noise

Sometimes when investigating disk issues, you’ll notice that many mounted snap partitions report 100% usage, like this:

## 🔍 What Does This Mean?

- These are read-only squashfs loop devices created by Snap packages.

- It's normal for them to show 100% usage — it doesn't mean your disk is full.

- However, having too many of them may:

- Clutter monitoring tools (e.g. df, iostat)

- Cause confusion during analysis

- Introduce additional background I/O if snaps auto-refresh frequently

## 💡 Tips

- You can list these mounts using:
```
df -h | grep /snap
```

To reduce clutter, consider using fewer Snap apps or switch to native/Flatpak alternatives if available:

```
sudo snap remove <unused-package>
```

---

## ✅ Conclusion

High disk I/O can cripple Linux system performance — especially on slower drives. By using tools like `iostat`, `iotop`, and Python-based visualizers, you can pinpoint the problem and take action.

Stay aware of hidden culprits like background processes and Snap mounts. With careful observation and tuning, your system can remain fast and responsive.

---

## 🔗 Let's Connect

If you found this article useful or have questions, feel free to reach out or follow me:

- 🌐 Personal Website: [farzan.us](https://farzan.us)
- 🧠 More posts on Dev.to: [@farzandev13](https://dev.to/farzandev13)

