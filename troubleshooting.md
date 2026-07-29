Issue:
kube-proxy crashed immediately after startup with:

failed complete: too many open files

Symptoms:
- Node Ready
- kube-proxy CrashLoopBackOff
- Ingress unavailable
- ArgoCD partially broken

Root cause:
Host inotify limits were too low:
fs.inotify.max_user_watches=65536
fs.inotify.max_user_instances=128

Fix:
Increase to:

fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=8192

Apply:
sudo sysctl --system
