# Lesson Expansion Checklist

ไฟล์นี้ใช้ติดตามงานเพิ่มรายละเอียดในแต่ละบทเรียน โดยจะทำทีละไฟล์ตามลำดับ และอัปเดตสถานะหลังทำเสร็จแต่ละไฟล์

## แนวทางการเพิ่มรายละเอียดในแต่ละบท

เป้าหมายคือทำให้แต่ละบทละเอียดขึ้น อ่านแล้วเข้าใจเหตุผลและลงมือทำได้จริง ไม่ใช่ใส่หัวข้อซ้ำ ๆ ทุกขั้นตอน ให้เลือกเพิ่มเฉพาะส่วนที่เหมาะกับบริบทของเนื้อหา

ประเด็นที่ควรพิจารณาเพิ่มเมื่อมีประโยชน์:

- ขั้นตอนนี้กำลังทำอะไร
- ทำเพื่ออะไร
- ทำไมต้องทำ
- ถ้าไม่ทำจะเกิดอะไรขึ้น
- ตัวอย่างที่ผิด
- ตัวอย่างที่ถูก
- วิธีทำหรือแนวทางตรวจสอบ

ถ้าขั้นตอนไหนตรงไปตรงมาอยู่แล้ว หรือการใส่ทุกประเด็นจะทำให้เอกสารรก ให้ข้ามประเด็นนั้นได้ โดยให้เน้นความละเอียดที่ช่วยลดความสับสน ลด error และทำให้ผู้อ่าน debug เองได้มากขึ้น

## สถานะไฟล์

| ลำดับ | ไฟล์ | สถานะ | หมายเหตุ |
|---:|---|---|---|
| 00 | `lessons/00-introduction.md` | เสร็จแล้ว | ปรับใหม่ให้เป็นบทนำละเอียดแบบธรรมชาติ พร้อมวิธีเรียน ข้อผิดพลาดที่พบบ่อย สิ่งที่ควรจด และ checklist ก่อนเริ่ม |
| 01 | `lessons/01-overview.md` | เสร็จแล้ว | เพิ่มภาพรวมสิ่งที่จะสร้าง role ของ VM, flow งาน DevOps, วิธีมองปัญหา และสิ่งที่ควรเข้าใจก่อนเริ่ม Lab 01 |
| 02 | `lessons/02-recommended-specs-and-network.md` | เสร็จแล้ว | เพิ่มเหตุผลการเลือกสเปก VM, NAT/Host-only, static IP, ตัวอย่าง network ผิด/ถูก และข้อควรระวังเรื่อง OS/user |
| 03 | `lessons/03-devops-terms.md` | เสร็จแล้ว | เพิ่มคำศัพท์เพิ่มเติมครอบคลุม Linux, Network, Docker, CI/CD, Kubernetes, Monitoring, Security และ Backup พร้อมคำที่มักสับสน |
| 04 | `lessons/04-lab-01-vm-and-ubuntu-server.md` | เสร็จแล้ว | เพิ่มรายละเอียดติดตั้ง VMware, ดาวน์โหลด Ubuntu Server ISO, ติดตั้ง OS, สร้าง VM, hostname, static IP, netplan, /etc/hosts และ SSH test |
| 05 | `lessons/05-lab-02-linux-ssh-service-log.md` | เสร็จแล้ว | เพิ่มบริบทคำสั่ง Linux, resource, process, systemctl, journalctl, permission, SSH log test และแนวทาง debug |
| 06 | `lessons/06-lab-03-network-dns-port-firewall.md` | เสร็จแล้ว | เพิ่มรายละเอียดการตรวจ IP/route/DNS/port, Nginx test, UFW, การแปลผล และ flow debug network |
| 07 | `lessons/07-lab-04-nginx-reverse-proxy-manual-deploy.md` | เสร็จแล้ว | เพิ่มรายละเอียด manual deploy, Node.js API, systemd service, Nginx reverse proxy, request flow, log และ debug |
| 08 | `lessons/08-lab-05-docker-basics.md` | เสร็จแล้ว | เพิ่มรายละเอียด Docker install, image/container flow, Dockerfile, .dockerignore, build/run, port mapping, logs และ debug |
| 09 | `lessons/09-lab-06-docker-compose-fullstack-app.md` | เสร็จแล้ว | เพิ่มรายละเอียด fullstack request flow, service DNS, env, volume, depends_on, Nginx route, testing และ Compose debug |
| 10 | `lessons/10-lab-07-private-container-registry.md` | เสร็จแล้ว | เพิ่มรายละเอียด registry flow, insecure registry, tag/push/pull, error ที่พบบ่อย และการตรวจสอบ |
| 11 | `lessons/11-lab-08-git-flow-release-version.md` | เสร็จแล้ว | เพิ่มรายละเอียด branch flow, semantic versioning, Git tag, release/rollback, Docker image tag และการตรวจสอบ |
| 12 | `lessons/12-lab-09-gitlab-ci-cd.md` | เสร็จแล้ว | เพิ่มรายละเอียด GitLab/Runner, pipeline flow, .gitlab-ci.yml, variables, registry, testing และ debug pipeline |
| 13 | `lessons/13-lab-10-terraform-iac.md` | เสร็จแล้ว | เพิ่มรายละเอียด Terraform workflow, provider/resource/state, init/plan/apply/destroy, state safety และการตรวจสอบ |
| 14 | `lessons/14-lab-11-prometheus-grafana-monitoring.md` | เสร็จแล้ว | เพิ่มรายละเอียด exporter, scrape flow, Prometheus config, Grafana datasource/dashboard, query และ debug target down |
| 15 | `lessons/15-lab-12-loki-promtail-logging.md` | เสร็จแล้ว | เพิ่มรายละเอียด Loki/Promtail flow, config, labels, Grafana datasource, LogQL, test log และ debug log ไม่เข้า |
| 16 | `lessons/16-lab-13-kubernetes-kubeadm.md` | เสร็จแล้ว | เพิ่มรายละเอียด kubeadm cluster, swap/kernel/sysctl, containerd, init/join, CNI, verification และ debug node NotReady |
| 17 | `lessons/17-lab-14-deploy-app-to-kubernetes.md` | เสร็จแล้ว | เพิ่มรายละเอียด Namespace, Deployment YAML, probe, resources, Service/endpoints, scale, rollout/rollback และ debug pod |
| 18 | `lessons/18-lab-15-ingress-configmap-secret-storage.md` | เสร็จแล้ว | เพิ่มรายละเอียด Ingress flow, ConfigMap/Secret usage, host/path, PV/PVC, hostPath caveat และ debug |
| 19 | `lessons/19-lab-16-helm-chart.md` | เสร็จแล้ว | เพิ่มรายละเอียด Helm chart/release/revision, values/templates, lint/template/dry-run, env values, upgrade/rollback และ debug |
| 20 | `lessons/20-lab-17-devsecops-trivy.md` | เสร็จแล้ว | เพิ่มรายละเอียด DevSecOps flow, Trivy image/fs scan, severity gate, CI example, hardening, image hygiene และข้อจำกัด scanner |
| 21 | `lessons/21-lab-18-backup-restore-dr.md` | เสร็จแล้ว | เพิ่มรายละเอียด backup/restore PostgreSQL, restore test, Kubernetes YAML backup, RPO/RTO, backup storage และ DR checklist |
| 22 | `lessons/22-final-project.md` | เสร็จแล้ว | เพิ่มรายละเอียด scope, phase การทำงาน, acceptance criteria, demo script, README requirement และข้อผิดพลาดที่ควรหลีกเลี่ยง |
| 23 | `lessons/23-completion-checklist.md` | เสร็จแล้ว | เพิ่มหลักฐานที่ควรมีในแต่ละหมวด, backup/restore checklist และเกณฑ์ผ่านแบบ practical |
| 24 | `lessons/24-troubleshooting.md` | เสร็จแล้ว | ขยายเป็น runbook troubleshooting ครอบคลุม SSH, Docker, Nginx, GitLab, Registry, Kubernetes, Ingress, Monitoring, Logging และ Terraform |
| 25 | `lessons/25-appendix-commands.md` | ยังไม่ทำ | ภาคผนวกคำสั่ง |
| 26 | `lessons/26-next-steps.md` | ยังไม่ทำ | แนวทางเรียนต่อ |
| 27 | `lessons/27-final-summary.md` | ยังไม่ทำ | สรุปสุดท้าย |

## Log การทำงาน

- ขยายรายละเอียด `lessons/00-introduction.md` แล้ว
- ปรับ `lessons/00-introduction.md` ใหม่ตามแนวทางว่าให้เพิ่มรายละเอียดเฉพาะจุดที่ควรใส่ ไม่ใช้ template บังคับทุกหัวข้อ
- ขยายรายละเอียด `lessons/01-overview.md` แล้ว
- ขยายรายละเอียด `lessons/02-recommended-specs-and-network.md` แล้ว
- ขยายรายละเอียด `lessons/03-devops-terms.md` แล้ว
- เพิ่มคำศัพท์เพิ่มเติมใน `lessons/03-devops-terms.md` ตามคำขอ
- ขยายรายละเอียด `lessons/04-lab-01-vm-and-ubuntu-server.md` แล้ว
- เพิ่มขั้นตอนติดตั้ง VMware, ดาวน์โหลด Ubuntu Server ISO และติดตั้ง OS ใน `lessons/04-lab-01-vm-and-ubuntu-server.md`
- ขยายรายละเอียด `lessons/05-lab-02-linux-ssh-service-log.md` แล้ว
- ขยายรายละเอียด `lessons/06-lab-03-network-dns-port-firewall.md` แล้ว
- ขยายรายละเอียด `lessons/07-lab-04-nginx-reverse-proxy-manual-deploy.md` แล้ว
- ขยายรายละเอียด `lessons/08-lab-05-docker-basics.md` แล้ว
- ขยายรายละเอียด `lessons/09-lab-06-docker-compose-fullstack-app.md` แล้ว
- ขยายรายละเอียด `lessons/10-lab-07-private-container-registry.md` แล้ว
- ขยายรายละเอียด `lessons/11-lab-08-git-flow-release-version.md` แล้ว
- ขยายรายละเอียด `lessons/12-lab-09-gitlab-ci-cd.md` แล้ว
- ขยายรายละเอียด `lessons/13-lab-10-terraform-iac.md` แล้ว
- ขยายรายละเอียด `lessons/14-lab-11-prometheus-grafana-monitoring.md` แล้ว
- ขยายรายละเอียด `lessons/15-lab-12-loki-promtail-logging.md` แล้ว
- ขยายรายละเอียด `lessons/16-lab-13-kubernetes-kubeadm.md` แล้ว
- ขยายรายละเอียด `lessons/17-lab-14-deploy-app-to-kubernetes.md` แล้ว
- ขยายรายละเอียด `lessons/18-lab-15-ingress-configmap-secret-storage.md` แล้ว
- ขยายรายละเอียด `lessons/19-lab-16-helm-chart.md` แล้ว
- ขยายรายละเอียด `lessons/20-lab-17-devsecops-trivy.md` แล้ว
- ขยายรายละเอียด `lessons/21-lab-18-backup-restore-dr.md` แล้ว
- ขยายรายละเอียด `lessons/22-final-project.md` แล้ว
- ขยายรายละเอียด `lessons/23-completion-checklist.md` แล้ว
- ขยายรายละเอียด `lessons/24-troubleshooting.md` แล้ว
