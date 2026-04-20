
**Author:** Samiksha Jayanth <PES1UG24CS414>

## Build & Run
```bash
sudo apt install -y gcc build-essential libssl-dev
export PES_AUTHOR="Samiksha Jayanth <PES1UG24CS414>"
make all
./pes init
./pes add <file>
./pes commit -m "message"
./pes log
```

## Analysis Questions

### Q5.1 — How would you implement `pes checkout <branch>`?
Read `.pes/HEAD` and update it to `ref: refs/heads/<branch>`. Read the target branch's commit object, get its tree hash, walk the tree recursively and restore every file to the working directory. This is complex because you must delete files present in the current branch but not in the target, handle conflicts with locally modified files, and do all this atomically to avoid a half-applied checkout.

### Q5.2 — How would you detect a dirty working directory before checkout?
For each file tracked in the index, compare its current `mtime` and `size` against the stored index values. If they differ, recompute the blob hash and compare it to the index hash. If any file differs, the working directory is dirty and checkout should be refused.

### Q5.3 — What happens to commits made in detached HEAD state?
New commits are written to the object store but no branch ref is updated — only HEAD (containing the raw hash) advances. If you switch away, those commits become unreachable from any branch. Recovery is possible if you remember the hash and create a new branch pointing to it before garbage collection removes it.

### Q6.1 — Describe a garbage collection algorithm for PES-VCS
Start from all branch refs in `.pes/refs/heads/`. BFS through every reachable commit, following parent links, then each commit's tree, then each tree's blobs. Collect all reachable hashes into a set. Walk all files under `.pes/objects/`, and delete any whose hash is not in the reachable set. For 100K commits with ~3 objects each across 50 branches, expect to visit roughly 300,000–500,000 objects.

### Q6.2 — What race condition exists during GC and how is it avoided?
GC marks objects as unreachable at time T. Meanwhile a commit in progress writes a blob at T+1 but hasn't yet written the tree/commit referencing it. GC deletes that blob at T+2. The commit then writes a tree pointing to a now-deleted blob — corruption. Git avoids this by never deleting objects newer than 2 weeks, and by using lock files so GC and commit cannot run simultaneously.
EOF