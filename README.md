# Staging_area — พื้นที่ทำงานและกู้คืนสำรองของทุกโปรเจกต์

**พื้นที่นี้ไม่ใช่รีโปโปรเจกต์หลักหรือที่ปล่อยผลิตภัณฑ์**. ใช้ได้เมื่อได้รับอนุญาตให้เก็บสำเนางาน, Diff, Tests, Integration, Recovery ใน Branch เฉพาะโปรเจกต์ และนำงานที่ตรวจจริงกลับไป `PROJECT_REPO` ก่อน Merge.

## ลิงก์บังคับ 3 แห่งสำหรับทุกงาน

1. **PROJECT_REPO:** URL เต็มของเกม/แอป/โครงการที่กำลังทำจริง — **ต้องระบุจาก AP-ROLE ของงานนั้น** ห้ามใช้ชื่อเกมก่อนหน้าโดยอัตโนมัติ
2. **ORG_WORKSPACE_REPO:** https://github.com/tanawutmalirum29-ux/ai-organization-hub — อ่าน [RULES.md](https://github.com/tanawutmalirum29-ux/ai-organization-hub/blob/main/RULES.md) (โครงสร้างองค์กรและ Backup/Runbook §§2, 5.1–5.3) และ [COMMANDS.md](https://github.com/tanawutmalirum29-ux/ai-organization-hub/blob/main/COMMANDS.md)
3. **STAGING_REPO:** https://github.com/tanawutmalirum29-ux/Staging_area — ใช้ Branch `stage/<project>/<mission-or-issue>` เมื่อมีสิทธิ์

**ห้าม Merge โค้ดเกมเข้า `Staging_area/main`.** `main` ของรีโปนี้เป็นคู่มือกลางของพื้นที่สำรองเท่านั้น. ห้ามเก็บ Token/Secrets/ข้อมูลผู้ใช้/เซฟจริงที่ไม่มีสิทธิ์เผยแพร่.

## โครงสร้างที่ใช้งาน

OWNER → PROJECT LEAD/CEO → ENGINEER/CREATOR/QA โดยตรง; Coordinator เป็นทางเลือกเมื่องานหลายส่วนและไม่เป็นด่านบังคับ. การสำรองไฟล์ไม่ได้เพิ่มสายอนุมัติ; Worker's ผลงานต้องกลับไปหา PROJECT LEAD และ QA ตามเกณฑ์โปรเจกต์.

## วิธีใช้แบบตรวจสอบได้

1. เปิด AP-ROLE ตรวจ `PROJECT_REPO`, Issue/Role/`REPORTS_TO`, Source Branch + **full SHA**, `EDITABLE_PATHS`, สิทธิ์เฉพาะการกระทำ, เหตุผลที่ต้องย้ายพื้นที่; ตรวจว่างานไม่ได้ถูก explicit-denied ด้านสิทธิ์/ความปลอดภัย.
2. **ทางแรก:** ทำงานใน Branch โปรเจกต์หลักหรือสำเนาท้องถิ่นก่อน ถ้าทำได้. พิจารณารีโปนี้เฉพาะกรณีปัญหาทางเทคนิค/Backup ที่ใช้ได้และไม่ละเมิดเงื่อนไขต้นทาง. อย่าสลับรีโปเพื่อหลบการปฏิเสธเดิม.
3. สร้าง Branch แยก `stage/<project>/<issue>` จากค่าที่มีจริง; วางเฉพาะ Source/Patch/Tests ที่ได้รับอนุญาต ไม่ปะปนโปรเจกต์อื่น.
4. บันทึก `STAGING_MANIFEST.md` ตามแบบด้านล่างและ Checksum. Commit/PR เฉพาะ Branch ที่เป็นของงานนี้, ไม่สร้างการเขียนซ้ำ/Force Push เพื่อลบงานคนอื่น.
5. รัน Tests/Build/Browser QA จริงเท่าที่เหมาะกับโปรเจกต์, แยก `PASS`, `FAIL` และ `NOT_RUN`. Syntax/Synthetic Test ผ่านไม่ได้แปลว่า Product CI ผ่าน.
6. เมื่อโปรเจกต์หลักใช้ได้ตามสิทธิ์ ให้ตรวจ latest `main` SHA, นำ Source/Diff กลับ Branch ใน `PROJECT_REPO`, เปิด PR, ให้ PROJECT LEAD ตรวจร่วมกับ QA อิสระตามความเสี่ยง แล้ว Merge เฉพาะ `PROJECT_REPO/main` หลัง Gate ผ่าน.
7. เมื่อส่งมอบแล้ว ตรวจ Restore point ยังใช้งานได้ จากนั้นทำ `PF` เฉพาะ Branch/ไฟล์ชั่วคราวของงานนี้, ไม่ลบ Branch/ไฟล์ของโปรเจกต์อื่นหรือของภารกิจที่ยัง HOLD.

## แบบฟอร์ม STAGING_MANIFEST.md

```yaml
PROJECT_REPO: https://github.com/<owner>/<project>
ORG_WORKSPACE_REPO: https://github.com/tanawutmalirum29-ux/ai-organization-hub
STAGING_REPO: https://github.com/tanawutmalirum29-ux/Staging_area
TASK_ISSUE: "#<issue>"
ROLE: "<role>"
REPORTS_TO: "PROJECT_LEAD #<lead-issue>"
SOURCE_BRANCH: "<branch>"
SOURCE_SHA: "<full-git-sha>"
TARGET_BRANCH: "stage/<project>/<issue>"
TARGET_SHA: "<full-git-sha-or-NOT_YET>"
EDITABLE_PATHS: ["<path>"]
CHANGED_PATHS: ["<path>"]
DEPENDENCIES: ["<PR/SHA>"]
BACKUP_REF: "<git-ref-or-local-copy>"
SHA256: "<hash-or-NOT_YET>"
TEST_RESULTS: "PASS|FAIL|NOT_RUN with commands and logs"
RESTORE_STEPS: "verify checksum; restore to disposable branch; diff; tests; PR"
CLEANUP_AFTER: "post-merge CI/QA confirmed and no active consumers"
```

## Runbook สั้นเมื่อเกิดปัญหา

| สาเหตุ | ทำต่อได้อย่างไร |
|---|---|
| Timeout, DNS, Clone หรือ GitHub 5xx | เก็บ error จริง, ตรวจ HEAD/Network, Retry ครั้งเดียวเมื่อเหมาะสม, ใช้สำเนาท้องถิ่น/connector แบบอ่านที่ได้รับอนุญาต |
| 429 / Rate limit | ดู `Retry-After`, อย่ายิงซ้ำถี่, เช็ก write เดิมว่าถูกทำแล้วหรือไม่ |
| SHA stale / PR conflict | ตรวจ Branch HEAD ใหม่, เปรียบ Diff และเจ้าของไฟล์, ใช้แผน Rebase/Integration ตามสิทธิ์ |
| CI/Build/Browser Fail | อ่าน failed job logs, แก้ที่ต้นเหตุ, อย่าข้าม Assertions, ส่ง QA ตรวจ exact SHA |
| Save/ข้อมูลผิด/ไฟล์แพตช์เขียนครึ่งทาง | Snapshot/Manifest/Checksum ก่อน, dry-run, backup+rollback ในสำเนา, ทดสอบ restore ก่อนนำกลับ |
| 401/403 หรือ explicit safety denial | หยุด **การเขียนเดียวกัน** ที่ถูกปฏิเสธ; ห้ามเปลี่ยนบทบาท/รีโป/API เพื่อหลบ. ทำ read-only/local work ที่อนุญาตหรือให้ผู้มีสิทธิ์จัดการ |
| Remote ทั้งหมดที่อนุญาตยังใช้ไม่ได้ | ZIP เป็นทางสุดท้าย มี Patch, 3 URLs, Source SHA, Checksum, Dry-run, Restore และผลทดสอบตามจริง; ไม่ให้ OWNER คัดลอกไฟล์จำนวนมาก |

**ดูขั้นตอนฉบับเต็ม:** [ai-organization-hub/RULES.md](https://github.com/tanawutmalirum29-ux/ai-organization-hub/blob/main/RULES.md) และ [ROLE-HANDOFF](https://github.com/tanawutmalirum29-ux/ai-organization-hub/blob/main/appointments/ROLE-HANDOFF.md).
