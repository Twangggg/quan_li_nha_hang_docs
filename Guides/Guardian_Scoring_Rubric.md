# Guardian Scoring Rubric v2.0

## Mục đích

Tài liệu này định nghĩa cách tính điểm Compliance, các ngưỡng pass/fail, và action tương ứng với từng mức severity. Đây là **nguồn sự thật duy nhất** cho scoring trong hệ thống FoodHub Guardian.

---

## Pass/Fail Thresholds

| Score     | Status               | Action                                        |
| --------- | -------------------- | --------------------------------------------- |
| ≥ 80/100  | ✅ **COMPLIANT**     | Có thể merge sau review thông thường          |
| 60–79/100 | ⚠️ **NEEDS WORK**    | Cần fix HIGH violations trước merge           |
| < 60/100  | ❌ **NON-COMPLIANT** | Block merge — cần fix toàn bộ CRITICAL + HIGH |

---

## Severity → Action Mapping

| Severity   | Màu | Định nghĩa                                                       | Action bắt buộc                                 |
| ---------- | --- | ---------------------------------------------------------------- | ----------------------------------------------- |
| `CRITICAL` | 🔴  | Vi phạm ảnh hưởng production an toàn, bảo mật, và data integrity | **Block merge ngay.** Fix trước khi review tiếp |
| `HIGH`     | 🟠  | Vi phạm ảnh hưởng quality và maintainability nghiêm trọng        | **Required fix** trước khi merge                |
| `MEDIUM`   | 🟡  | Vi phạm ảnh hưởng code quality, không block production ngay      | Fix trong sprint hiện tại                       |
| `LOW`      | 🟢  | Style, naming, non-critical convention                           | Track trong backlog, không block                |

---

## Per-Skill Scoring Rubrics

### FFA-FLW (Flow Analyzer) — 100 điểm

| Rule                            | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                      |
| ------------------------------- | --- | ---------------- | ---------------------------------------------- |
| R1: Validation ở đầu            | 25  | CRITICAL         | FluentValidation được gọi trước mọi logic khác |
| R2: Log Start + End             | 20  | HIGH             | Có cả log đầu và log cuối Handler              |
| R3: Transaction wrapper         | 25  | CRITICAL         | Tất cả multi-write trong Begin/Commit/Rollback |
| R4: Business logic trong Entity | 15  | HIGH             | Handler không gán property Entity trực tiếp    |
| R5: Mapping/Return tách biệt    | 15  | MEDIUM           | Mapping nằm ở bước cuối, sau save/commit       |

### FFA-TXG (Transaction Guard) — 100 điểm

| Rule                                  | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                       |
| ------------------------------------- | --- | ---------------- | ----------------------------------------------- |
| TX-1: IUnitOfWork                     | 25  | CRITICAL         | Dùng IUnitOfWork, không gọi DbContext trực tiếp |
| TX-2: BeginTransactionAsync           | 25  | CRITICAL         | Có BeginTransactionAsync trước write đầu tiên   |
| TX-3: CommitTransactionAsync          | 20  | HIGH             | Có Commit sau khi tất cả write thành công       |
| TX-4: RollbackTransactionAsync        | 20  | CRITICAL         | Có Rollback trong catch block                   |
| TX-5: External call ngoài transaction | 10  | MEDIUM           | Email/API call không bọc trong transaction      |

### FFA-LOG (Logging Compliance) — 100 điểm

| Rule                        | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                 |
| --------------------------- | --- | ---------------- | ----------------------------------------- |
| LOG-1: Log Start            | 25  | HIGH             | LogInformation ở đầu Handle method        |
| LOG-2: Log End/Success      | 25  | HIGH             | LogInformation khi hoàn thành thành công  |
| LOG-3: No sensitive data    | 30  | CRITICAL         | Không có password/token/secret trong log  |
| LOG-4: Structured logging   | 15  | MEDIUM           | Dùng placeholder {} thay vì string concat |
| LOG-5: Error log on failure | 5   | MEDIUM           | LogWarning/LogError khi exception/failure |

### FFA-CAG (Clean Architecture Guard) — 100 điểm

| Rule                                 | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                                |
| ------------------------------------ | --- | ---------------- | -------------------------------------------------------- |
| CAG-1: Handler chỉ orchestrate       | 30  | CRITICAL         | Không có direct property assignment trong Handler        |
| CAG-2: State change trong Entity     | 30  | CRITICAL         | Entity có method cho mọi state change                    |
| CAG-3: Domain không depend App/Infra | 20  | CRITICAL         | Domain namespace không import Application/Infrastructure |
| CAG-4: Factory/Constructor pattern   | 10  | MEDIUM           | Entity có rõ ràng factory method hoặc constructor        |
| CAG-5: Value Objects                 | 10  | LOW              | Dùng VO cho concepts có validation (Email, Money)        |

### FFA-CTL (Controller Standard) — 100 điểm

| Rule                                      | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                           |
| ----------------------------------------- | --- | ---------------- | --------------------------------------------------- |
| CTL-1: Kế thừa ApiControllerBase          | 20  | HIGH             | `class XController : ApiControllerBase`             |
| CTL-2: Inject IMediator only              | 25  | CRITICAL         | Không inject Repository/Service trực tiếp           |
| CTL-3: Không có business if/else          | 25  | CRITICAL         | Action method chỉ gọi Mediator.Send()               |
| CTL-4: HandleResult cho response          | 20  | HIGH             | Dùng HandleResult(result) thay vì Ok()/BadRequest() |
| CTL-5: [Authorize] + ProducesResponseType | 10  | MEDIUM           | Attribute đầy đủ cho bảo mật và Swagger             |

### FFA-ACV (API Contract Validator) — 100 điểm

| Rule                                        | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                    |
| ------------------------------------------- | --- | ---------------- | -------------------------------------------- |
| ACV-1: FluentValidation cho required fields | 30  | CRITICAL         | Mọi required field có RuleFor                |
| ACV-2: MaximumLength cho string             | 20  | HIGH             | Mọi string field có .MaximumLength()         |
| ACV-3: Proper Type                          | 20  | HIGH             | Guid/Enum/decimal thay vì string/int/double  |
| ACV-4: XML Documentation                    | 15  | MEDIUM           | /// <summary> cho class và public properties |
| ACV-5: Response không expose internals      | 15  | MEDIUM           | Không trả PasswordHash, Token trong response |

### FFA-SEC (Security Validation) — 100 điểm

| Rule                              | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                               |
| --------------------------------- | --- | ---------------- | ------------------------------------------------------- |
| SEC-1: [Authorize] attribute      | 30  | CRITICAL         | Mọi endpoint cần bảo vệ có [Authorize]                  |
| SEC-2: Clean response DTO         | 25  | CRITICAL         | Response không trả password/token/secret                |
| SEC-3: CSRF protection            | 20  | HIGH             | Endpoint mutate state có CSRF protection (Cookie auth)  |
| SEC-4: Fine-grained authorization | 15  | HIGH             | Ownership check trong Handler cho resource-level access |
| SEC-5: No stack trace in response | 10  | HIGH             | Exception middleware xử lý, không trả detail ra ngoài   |

### FFA-PERF (Performance Risk) — 100 điểm

| Rule                               | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                         |
| ---------------------------------- | --- | ---------------- | ------------------------------------------------- |
| PERF-1: Pagination trên list query | 30  | CRITICAL         | Có Skip+Take hoặc Cursor — không ToListAsync() mở |
| PERF-2: AsNoTracking trên read     | 25  | HIGH             | Read-only query có .AsNoTracking()                |
| PERF-3: Không có N+1               | 25  | HIGH             | Không có DB call trong foreach/loop               |
| PERF-4: Projection .Select()       | 10  | MEDIUM           | Dùng .Select() thay vì load toàn Entity           |
| PERF-5: Không SELECT \*            | 10  | MEDIUM           | Chỉ lấy columns cần thiết                         |

### FFA-ERR (Error Handling) — 100 điểm

| Rule                                | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                                           |
| ----------------------------------- | --- | ---------------- | ------------------------------------------------------------------- |
| ERR-1: Custom Exception             | 25  | CRITICAL         | Dùng BusinessException/NotFoundException, không throw raw Exception |
| ERR-2: ExceptionMiddleware Coverage | 25  | CRITICAL         | Mọi exception type đều có mapping trong middleware                  |
| ERR-3: No Stack Trace in Production | 20  | CRITICAL         | Production response không chứa stack trace, internal detail         |
| ERR-4: Consistent ErrorResponse     | 15  | HIGH             | Mọi exception trả về dạng ErrorResponse(statusCode, message)        |
| ERR-5: No Handler Catch-All         | 15  | HIGH             | Handler không catch tổng quát rồi swallow — để middleware xử lý     |

### FFA-TST (Backend Test Compliance) — 100 điểm

| Rule                           | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                               |
| ------------------------------ | --- | ---------------- | ------------------------------------------------------- |
| TST-1: Test file tồn tại       | 25  | CRITICAL         | Command Handler có {HandlerName}Tests.cs tương ứng      |
| TST-2: Happy Path test         | 25  | CRITICAL         | Có ít nhất 1 test cho success scenario chính            |
| TST-3: Error Path test         | 20  | HIGH             | Test cho validation fail, not found, business rule fail |
| TST-4: Mock pattern            | 15  | HIGH             | Mock interface (IUnitOfWork), không mock concrete class |
| TST-5: AAA + Assertion quality | 15  | MEDIUM           | Arrange-Act-Assert rõ ràng, assertion có ý nghĩa        |

---

## FGO Overall Score Formula

```
Command Handler:
  Overall = (FLW_score × 0.25) + (TXG_score × 0.25) + (LOG_score × 0.15) + (CAG_score × 0.15) + (ERR_score × 0.10) + (TST_score × 0.10)

Query Handler:
  Overall = (FLW_score × 0.25) + (LOG_score × 0.15) + (PERF_score × 0.20) + (CAG_score × 0.20) + (TST_score × 0.20)

Controller:
  Overall = (CTL_score × 0.35) + (SEC_score × 0.30) + (ACV_score × 0.20) + (ERR_score × 0.15)

Mixed/PR:
  Overall = Average(all applicable file scores)
```

---

## Ví dụ Score Calculation

**Ví dụ: CreateOrderHandler (Command)**

| Skill       | Raw Score | Weight | Weighted                 |
| ----------- | --------- | ------ | ------------------------ |
| FFA-FLW     | 85/100    | 30%    | 25.5                     |
| FFA-TXG     | 100/100   | 30%    | 30.0                     |
| FFA-LOG     | 70/100    | 20%    | 14.0                     |
| FFA-CAG     | 90/100    | 20%    | 18.0                     |
| **Overall** |           |        | **87.5% → ✅ COMPLIANT** |

---

_Rubric version 2.0 — FoodHub Guardian System. Cập nhật khi có skill version mới._

---

## FFO Overall Score Formula (Frontend)

```
Component Review:
  Overall = (CMP_score × 0.35) + (PERF_score × 0.25) + (STR_score × 0.25) + (TST_score × 0.15)

Page Review:
  Overall = (CMP_score × 0.30) + (PERF_score × 0.25) + (STR_score × 0.35) + (TST_score × 0.10)

Service Review:
  Overall = (API_score × 0.70) + (TST_score × 0.30)

Hook Review:
  Overall = (API_score × 0.40) + (PERF_score × 0.30) + (TST_score × 0.30)

Auth Review:
  Overall = SEC_score × 1.00

Full Feature PR:
  Overall = (CMP_score × 0.20) + (API_score × 0.20) + (SEC_score × 0.20) + (PERF_score × 0.15) + (STR_score × 0.10) + (TST_score × 0.15)
```

---

## Per-Skill Scoring Rubrics (Frontend)

### FFE-CMP (Component Standard) — 100 điểm

| Rule                          | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                             |
| ----------------------------- | --- | ---------------- | ----------------------------------------------------- |
| CMP-1: Single Responsibility  | 25  | HIGH             | Component không quá 1 responsibility                  |
| CMP-2: Props TypeScript type  | 25  | CRITICAL         | Props có interface/type rõ ràng, không dùng `any`     |
| CMP-3: Client/Server boundary | 20  | HIGH             | `"use client"` chỉ khi thực sự cần hook/event handler |
| CMP-4: Component placement    | 15  | MEDIUM           | Reusable component trong `src/components/`            |
| CMP-5: Naming convention      | 15  | LOW              | PascalCase.tsx cho file, kebab-case cho folder        |

### FFE-API (API Integration) — 100 điểm

| Rule                            | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                                   |
| ------------------------------- | --- | ---------------- | ----------------------------------------------------------- |
| API-1: Service layer only       | 30  | CRITICAL         | API call trong `src/services/`, không fetch trong component |
| API-2: TypeScript response type | 25  | CRITICAL         | Response có type rõ ràng, không dùng `any`                  |
| API-3: Error handling           | 20  | HIGH             | try-catch hoặc .catch() cho mọi API call                    |
| API-4: Form validation          | 15  | HIGH             | Zod/Yup validate trước khi submit                           |
| API-5: Loading/Error state      | 10  | MEDIUM           | UI hiển thị loading và error cho user                       |

### FFE-SEC (Frontend Security) — 100 điểm

| Rule                             | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                              |
| -------------------------------- | --- | ---------------- | ------------------------------------------------------ |
| FSEC-1: No localStorage token    | 30  | CRITICAL         | Token/session dùng HttpOnly Cookie, không localStorage |
| FSEC-2: No console.log sensitive | 25  | CRITICAL         | Không console.log token, user data, password           |
| FSEC-3: Route guard              | 20  | CRITICAL         | Protected route có middleware redirect                 |
| FSEC-4: No exposed secrets       | 15  | CRITICAL         | NEXT*PUBLIC* không chứa secret/key                     |
| FSEC-5: Input sanitization       | 10  | HIGH             | URL params được sanitize trước khi render              |

### FFE-PERF (Frontend Performance) — 100 điểm

| Rule                            | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                          |
| ------------------------------- | --- | ---------------- | -------------------------------------------------- |
| FPERF-1: Server Component fetch | 25  | HIGH             | Data fetch trong Server Component, không useEffect |
| FPERF-2: No N+1 FE              | 25  | HIGH             | Không có API call trong loop                       |
| FPERF-3: next/image             | 20  | MEDIUM           | Dùng `next/image` thay vì `<img>` tag              |
| FPERF-4: useMemo                | 15  | MEDIUM           | Expensive computation có useMemo                   |
| FPERF-5: useCallback            | 15  | LOW              | Callback prop có useCallback                       |

### FFE-STR (Project Structure) — 100 điểm

| Rule                             | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                              |
| -------------------------------- | --- | ---------------- | ------------------------------------------------------ |
| STR-1: app/ routing only         | 25  | HIGH             | `app/` không chứa component logic > 50 lines           |
| STR-2: Naming convention         | 20  | MEDIUM           | PascalCase.tsx, useXxx.ts, kebab-case folder           |
| STR-3: Types in src/types/       | 20  | MEDIUM           | Types/interfaces không inline trong component          |
| STR-4: Services in src/services/ | 20  | HIGH             | Service functions không đặt trong app/ hay components/ |
| STR-5: No circular imports       | 15  | HIGH             | Components không import từ app/                        |

### FFE-TST (Frontend Test Compliance) — 100 điểm

| Rule                             | Max | Severity nếu = 0 | Điều kiện đạt điểm tối đa                             |
| -------------------------------- | --- | ---------------- | ----------------------------------------------------- |
| FTST-1: Test file tồn tại        | 25  | HIGH             | Feature component có {Name}.test.tsx tương ứng        |
| FTST-2: Render test              | 25  | HIGH             | Test verify component hiển thị đúng content           |
| FTST-3: Interaction test         | 20  | HIGH             | Test verify click, submit, input change đúng behavior |
| FTST-4: User-centric queries     | 15  | MEDIUM           | Dùng getByRole/getByText, không querySelector         |
| FTST-5: Error/Loading state test | 15  | MEDIUM           | Test error state và loading state (nếu có API call)   |
