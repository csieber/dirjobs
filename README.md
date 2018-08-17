# dirjobs - A simple file-based multi-worker, single-queue job queue

Provides a Python file/directory-based simple job queue with support for multiple workers and lazy synchronization. Designed to work over webdav or Windows share mounts without any locking mechanisms available.

The intended use case is for experimental set-ups in research.

Features:

  * File / Directory-based, no use of locks
  * Works over network mounts such as webdav or Windows shares
  * 4 job states: *waiting*, *running*, *done* and *failed*
  * Synchronization by waiting if another worker also claims the job
  * In case of conflict, the job gets assigned to the first worker in lexicographical order

Notes:

  * If sync fails, a job maybe executed by two workers in parallel
  * Sync time can be long over webdav. Only useful for long-running (> 5 minutes) tasks.

## QUICKSTART (one worker)

Create a folder for waiting jobs:

```bash
    mkdir -p jobs/00_waiting
```
    
Create empty jobs:

```bash
    touch jobs/00_waiting/job1.txt
    touch jobs/00_waiting/job2.txt
```
    
Afterwards run:

```python
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

## QUICKSTART (multiple workers)

Use the argument *wid* to create multiple workers. 

**Important**: Worker IDs have to be unique !

Example first worker:

```python
    dj = DirJobs("jobs/", wid="w1")
```
 
Example second worker:

```python
    dj = DirJobs("jobs/", wid="w2")
```

## Examples

### Filter jobs

Jobs can be filtered by a filter function with the signature *func(path, name) -> boolean*.

Example: Only select jobs with *tag* in their filename:

```python
    def job_filter(path, name):
        if "tag" in name:
            return True
        else:
            return False

    dj = DirJobs("jobs/", job_filter=job_filter)
```

### WebDAV mounts

Mount a webdav folder under your user account:

```bash
sudo mount.davfs -o uid=`whoami` https://YOUR_WEBDAV_URL/ WEBDAV/
```

Activate *worker_sync* and set a *sync_time* in seconds:

```python
dj = DirJobs(args.jobdir,
             wid="w1",
             worker_sync=True,
             sync_time=70)
```

  * *sync_time*: Set this time to the amount webdav approx. needs to sync the job files between the workers. 70s seems fine for default webdav configuration. If the sync fails, a job maybe executed by two workers at the same time.
