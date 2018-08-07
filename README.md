# dirjobs - A simple file-based multi-worker, single-queue job queue

Features:

  * File / Directory-based, no use of locks
  * Works over network mounts such as webdav or Windows shares
  * 4 job states: *waiting*, *running*, *done* and *failed*

## QUICKSTART

Create a folder for waiting jobs:

    mkdir -p jobs/00_waiting
    
Create empty jobs:

    touch jobs/00_waiting/job1.txt
    touch jobs/00_waiting/job2.txt
    
Afterwards run:


```
from dirjobs import DirJobs

dj = DirJobs("jobs/")

while True:
        
    job = dj.next_and_lock()
        
    if not job:
        print("No more jobs!")
        break
       
    print("Processing: %s" % job.path())
        
    job.done() # or job.failed()

```

The output should be:

```
Processing: jobs/01_running/w1.job1.txt
Processing: jobs/01_running/w1.job2.txt
No more jobs!
```