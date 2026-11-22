# API reference

This inventory expands every literal controller mapping, including aliases in annotation arrays. Java signatures below specify body types, query/path parameters and response types; the linked source defines the complete record fields. This is a source-verified contract reference, not an OpenAPI generator or a claim that every route was called during this documentation pass.

## Common contract

Requests and responses use JSON. Send the session cookie on all authenticated requests. Before POST/PUT obtain GET /api/auth/csrf and send its token using the returned headerName. Login/logout also require CSRF. Public means authentication is not required; it does not exempt mutation requests from CSRF.

SecurityConfig protects /api/admin/** (ADMIN), /api/technician/** (TECHNICIAN) and /api/student/** (STUDENT). Shared incident routes additionally validate ownership/team membership. All other API routes require an enabled authenticated user unless marked public below. Identity comes from the session, never an actor ID supplied by the client.

Default controller success is HTTP 200; incident creation and registration return 201. Bean/type validation generally returns 400, denied roles/participants 403, missing records 404 where explicitly handled, and invalid lifecycle/conflict operations 409. Some catalog/user lookups deliberately throw IllegalArgumentException and therefore return 400 rather than 404. Do not assume all missing resources have identical status codes. Session failure is 401 and CSRF failure is 403. Controller advice normally returns {"message":"..."}; infrastructure errors may differ.

[IncidentApiErrors.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/IncidentApiErrors.java) and [SecurityConfig.java](../bits-itsm/backend/src/main/java/com/bits/itsm/auth/SecurityConfig.java) define these boundaries. Error conditions below supplement these common failures; ordinary GETs have no body. Path variables are required and query parameters optional unless a default/annotation says otherwise.

## Request examples

Create a disabled custom service (ADMIN, POST /api/admin/catalog/services):

```json
{"code":"LIBRARY_PROXY","label":"Library Proxy","description":"Off-campus access to library resources","keywords":"library proxy,remote library access","criticality":"MEDIUM","enabled":false}
```

MEDIUM is an example administrator choice, not a recommended value for every service. Use PUT /api/admin/catalog/SERVICE/LIBRARY_PROXY with the CatalogEdit fields to activate it after configuration. Codes must be unique; activation without valid criticality fails. The response is the saved CatalogEntry. GET /api/incidents/options returns eligible service value/label pairs.

Student incident creation (POST /api/incidents):

```json
{"title":"Hostel Wi-Fi disconnects during online classes","description":"Several students on our hostel floor experience repeated Wi-Fi disconnections during online classes. Around 20 students are affected.","category":"NETWORK","affectedService":"CAMPUS_WIFI","urgency":"HIGH","affectedUsers":20}
```

These are reported hints. Actual configured evidence/model determines effective classification and priority; this example is not a guarantee against every possible administrator configuration. The response is IncidentData.View with status, effective priority, review flags, primary technician and deadlines.

Other concise bodies:

| Operation | JSON body |
|---|---|
| Technician availability | {"availability":"UNAVAILABLE"} |
| Priority override | {"priority":"P2","reason":"Teaching session is blocked for the whole lab."} |
| Manual escalation | {"level":"TEAM_LEAD","reason":"Specialist assistance is required to restore access."} |
| Staff message | {"content":"Please inspect the access point logs.","urgent":false} |
| Mark channel read | {"messageId":123} |
| Verify resolution | {"confirmed":true,"reason":"Connectivity is stable again."} |

These examples describe schemas; they are not records created by this task.

## Endpoint inventory

### AdminManagementController

Source: [AdminManagementController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/api/AdminManagementController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/catalog

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<CatalogEntry>` from `catalog`.
- **Failures:** Duplicate/invalid code, required fields, enabled service missing criticality, invalid keywords or catalog kind.

#### PUT /api/admin/catalog/{kind}/{code}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable String kind, @PathVariable String code, @Valid @RequestBody CatalogEdit request, Authentication auth`.
- **Output:** `CatalogEntry` from `updateCatalog`.
- **Failures:** Duplicate/invalid code, required fields, enabled service missing criticality, invalid keywords or catalog kind.

#### POST /api/admin/catalog/categories

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody NewCategory request,Authentication auth`.
- **Output:** `CatalogEntry` from `createCategory`.
- **Failures:** Duplicate/invalid code, required fields, enabled service missing criticality, invalid keywords or catalog kind.

#### POST /api/admin/catalog/services

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody NewService request, Authentication auth`.
- **Output:** `CatalogEntry` from `createService`.
- **Failures:** Duplicate/invalid code, required fields, enabled service missing criticality, invalid keywords or catalog kind.

#### GET /api/admin/classification-review

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `List<IncidentData.View>` from `classificationReview`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/classification-settings

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `ClassificationSettings` from `classificationSettings`.
- **Failures:** Common authentication, validation and lookup failures above.

#### PUT /api/admin/classification-settings

- **Access:** ADMIN.
- **Inputs:** `@RequestBody ClassificationSettings request, Authentication auth`.
- **Output:** `ClassificationSettings` from `updateClassificationSettings`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/email-outbox

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<NotificationDelivery>` from `outbox`.
- **Failures:** Common authentication, validation and lookup failures above.

#### POST /api/admin/incidents/{id}/classify

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody IncidentData.AdminCorrection request, Authentication auth`.
- **Output:** `IncidentData.View` from `classifyIncident`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/admin/reports

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `Map<String, Object>` from `reports`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/users

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<UserView>` from `users`.
- **Failures:** Duplicate email, required student ID/technician skill, self-disable or unresolved technician assignments.

#### POST /api/admin/users

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody NewUser request, Authentication auth`.
- **Output:** `UserView` from `create`.
- **Failures:** Duplicate email, required student ID/technician skill, self-disable or unresolved technician assignments.

#### PUT /api/admin/users/{id}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody UserEdit request, Authentication auth`.
- **Output:** `UserView` from `update`.
- **Failures:** Duplicate email, required student ID/technician skill, self-disable or unresolved technician assignments.

### ServicesController

Source: [ServicesController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/api/ServicesController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/audit

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<AuditRecord>` from `auditAll`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/audit/{incidentId}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long incidentId`.
- **Output:** `List<AuditRecord>` from `auditFor`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/escalations

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<EscalationRecord>` from `escalations`.
- **Failures:** Missing incident; mutation requires assignable incident and a higher level with reason.

#### GET /api/admin/incidents/{incidentId}/related

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long incidentId`.
- **Output:** `List<DuplicateMatch>` from `related`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/admin/knowledge

- **Access:** ADMIN.
- **Inputs:** `Authentication auth, @RequestParam(required = false) String search, @RequestParam(required = false) String category, @RequestParam(required = false) String service`.
- **Output:** `List<KnowledgeEntry>` from `adminKnowledge`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/sla/violations

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<SlaSnapshot>` from `slaViolations`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/incidents/{incidentId}/suggested-solutions

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long incidentId, Authentication auth`.
- **Output:** `List<Suggestion>` from `suggestedSolutions`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/notifications

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<InAppNotification>` from `myNotifications`.
- **Failures:** Another recipient’s/missing notification is not readable or mutable.

#### POST /api/notifications/{notificationId}/read

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `@PathVariable String notificationId, Authentication auth`.
- **Output:** `Map<String, Boolean>` from `markRead`.
- **Failures:** Another recipient’s/missing notification is not readable or mutable.

#### POST /api/notifications/read-all

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication auth`.
- **Output:** `Map<String, Integer>` from `markAllNotificationsRead`.
- **Failures:** Another recipient’s/missing notification is not readable or mutable.

#### GET /api/notifications/unread-count

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication auth`.
- **Output:** `Map<String, Long>` from `unread`.
- **Failures:** Another recipient’s/missing notification is not readable or mutable.

#### GET /api/technician/knowledge

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<KnowledgeEntry>` from `knowledge`.
- **Failures:** Common authentication, validation and lookup failures above.

### AssignmentController

Source: [AssignmentController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/assignment/AssignmentController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/assignment-preview/{id}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `AssignmentService.AssignmentPreview` from `preview`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/admin/assignment-strategy

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `Strategy` from `strategy`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### PUT /api/admin/assignment-strategy

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody Strategy request, Authentication auth`.
- **Output:** `Strategy` from `strategy`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/admin/collaboration-requests

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `List<RequestView>` from `collaborations`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/admin/collaboration-requests/{id}/approve

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `approveCollaboration`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/admin/collaboration-requests/{id}/reject

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `rejectCollaboration`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/admin/incidents/{id}/collaborators

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody TeamChange request,Authentication auth`.
- **Output:** `IncidentData.View` from `manage`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### PUT /api/admin/technicians/{id}/assignment-profile

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Profile request, Authentication auth`.
- **Output:** `WorkloadService.Workload` from `profile`.
- **Failures:** Missing/invalid availability, skills or capacity; unknown technician.

#### GET /api/admin/technicians/workloads

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `List<WorkloadService.Workload>` from `workloads`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/transfer-requests

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `List<RequestView>` from `transfers`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/admin/transfer-requests/{id}/approve

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `approveTransfer`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/admin/transfer-requests/{id}/reject

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `rejectTransfer`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/incidents/{id}/assign

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody IncidentData.Assignment request, Authentication auth`.
- **Output:** `IncidentData.View` from `assign`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/incidents/{id}/auto-assign

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `IncidentData.View` from `auto`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/incidents/{id}/collaboration-requests

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Collaboration request, Authentication auth`.
- **Output:** `RequestView` from `collaborate`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/incidents/{id}/eligible-technicians

- **Access:** ADMIN or authorized technician; service checks.
- **Inputs:** `@PathVariable Long id, @RequestParam(defaultValue = "false") boolean collaboration, @RequestParam(required = false) TechnicianSkill skill, Authentication auth`.
- **Output:** `List<WorkloadService.Workload>` from `eligible`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/incidents/{id}/transfer-requests

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Transfer request, Authentication auth`.
- **Output:** `RequestView` from `transfer`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/technician/availability

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `java.util.Map<String, Object>` from `availability`.
- **Failures:** Missing/invalid availability, skills or capacity; unknown technician.

#### PUT /api/technician/availability

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `@Valid @RequestBody AvailabilityUpdate request, Authentication auth`.
- **Output:** `java.util.Map<String, Object>` from `updateAvailability`.
- **Failures:** Missing/invalid availability, skills or capacity; unknown technician.

#### GET /api/technician/collaboration-requests

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<RequestView>` from `myCollaborations`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/technician/collaboration-requests/{id}/cancel

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `cancelCollaboration`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/technician/team-incidents

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<IncidentData.View>` from `team`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/technician/transfer-requests

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<RequestView>` from `myTransfers`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### POST /api/technician/transfer-requests/{id}/cancel

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Review request, Authentication auth`.
- **Output:** `RequestView` from `cancelTransfer`.
- **Failures:** Ineligible destination, wrong owner/status, missing reason, completed request or capacity/skill conflict.

#### GET /api/technician/workload

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `WorkloadService.Workload` from `workload`.
- **Failures:** Common authentication, validation and lookup failures above.

### AuthController

Source: [AuthController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/auth/AuthController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/auth/csrf

- **Access:** Public; CSRF on writes.
- **Inputs:** `CsrfToken token`.
- **Output:** `Map<String, String>` from `csrf`.
- **Failures:** Common authentication, validation and lookup failures above.

#### POST /api/auth/login

- **Access:** Public; CSRF on writes.
- **Inputs:** `@Valid @RequestBody LoginRequest credentials, HttpServletRequest request, HttpServletResponse response`.
- **Output:** `Account` from `login`.
- **Failures:** Invalid credentials, disabled account, wrong selected role after credential verification.

#### GET /api/auth/me

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication authentication`.
- **Output:** `Account` from `me`.
- **Failures:** Common authentication, validation and lookup failures above.

#### POST /api/auth/register

- **Access:** Public; CSRF on writes.
- **Inputs:** `@RequestBody Registration input`.
- **Output:** `ResponseEntity<Map<String, String>>` from `register`.
- **Failures:** Email/password/name validation; duplicate email (409).

### PasswordController

Source: [PasswordController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/auth/PasswordController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### POST /api/admin/users/{id}/password-reset

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,Authentication auth`.
- **Output:** `ResponseEntity<PasswordRecovery.ResetCode>` from `initiate`.
- **Failures:** Invalid/expired/consumed code, wrong current password, password policy or confirmation mismatch; initiation requires enabled account.

#### POST /api/auth/change-password

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `@RequestBody Input input,Authentication auth`.
- **Output:** `Map<String,String>` from `change`.
- **Failures:** Invalid/expired/consumed code, wrong current password, password policy or confirmation mismatch; initiation requires enabled account.

#### GET /api/auth/password-capabilities

- **Access:** Public; CSRF on writes.
- **Inputs:** None.
- **Output:** `Map<String,Boolean>` from `capabilities`.
- **Failures:** Invalid/expired/consumed code, wrong current password, password policy or confirmation mismatch; initiation requires enabled account.

#### POST /api/auth/reset-password

- **Access:** Public; CSRF on writes.
- **Inputs:** `@RequestBody Input input`.
- **Output:** `Map<String,String>` from `reset`.
- **Failures:** Invalid/expired/consumed code, wrong current password, password policy or confirmation mismatch; initiation requires enabled account.

### HealthController

Source: [HealthController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/HealthController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/health

- **Access:** Public; CSRF on writes.
- **Inputs:** None.
- **Output:** `Map<String, String>` from `health`.
- **Failures:** Common authentication, validation and lookup failures above.

### ConversationController

Source: [ConversationController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ConversationController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/incidents/{id}/internal-messages

- **Access:** ADMIN or primary/approved collaborator; urgent send ADMIN only.
- **Inputs:** `@PathVariable Long id,Authentication auth`.
- **Output:** `IncidentConversation.StaffConversation` from `listInternal`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

#### POST /api/incidents/{id}/internal-messages

- **Access:** ADMIN or primary/approved collaborator; urgent send ADMIN only.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody InternalMessage message,Authentication auth`.
- **Output:** `IncidentConversation.StaffConversation` from `sendInternal`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

#### POST /api/incidents/{id}/internal-messages/read

- **Access:** ADMIN or primary/approved collaborator; urgent send ADMIN only.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Read read,Authentication auth`.
- **Output:** `java.util.Map<String,Boolean>` from `readInternal`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

#### GET /api/incidents/{id}/messages

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long id,Authentication auth`.
- **Output:** `IncidentConversation.Conversation` from `list`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

#### POST /api/incidents/{id}/messages

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Message message,Authentication auth`.
- **Output:** `IncidentConversation.Conversation` from `send`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

#### POST /api/incidents/{id}/messages/read

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Read read,Authentication auth`.
- **Output:** `java.util.Map<String,Boolean>` from `read`.
- **Failures:** Nonparticipant/channel denial, blank/oversized text, invalid read cursor; urgent staff write denied for non-admin.

### HistoricalResolutionController

Source: [HistoricalResolutionController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/HistoricalResolutionController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/incidents/{id}/resolution-estimate

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `Map<String,Object>` from `estimate`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

### IncidentController

Source: [IncidentController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/IncidentController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/incidents

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication auth, jakarta.servlet.http.HttpServletRequest request, @RequestParam(required = false) String search, @RequestParam(required = false) IncidentStatus status, @RequestParam(required = false) Incident.Category category, @RequestParam(required = false) Incident.Priority priority`.
- **Output:** `List<View>` from `list`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents

- **Access:** STUDENT.
- **Inputs:** `@Valid @RequestBody Create request, Authentication auth`.
- **Output:** `View` from `create`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/incidents/{id}

- **Access:** Authorized incident participant / ADMIN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `View` from `get`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/acknowledge

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `View` from `acknowledge`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/classify

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Classification request, Authentication auth`.
- **Output:** `View` from `classify`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/close

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `View` from `close`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/disposition

- **Access:** Reporter for WITHDRAWN; ADMIN for INVALID/DUPLICATE.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Disposition request, Authentication auth`.
- **Output:** `View` from `disposition`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/edit

- **Access:** Reporting STUDENT.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Edit request, Authentication auth`.
- **Output:** `View` from `edit`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/feedback

- **Access:** Reporting STUDENT.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Feedback request, Authentication auth`.
- **Output:** `View` from `feedback`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/information

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Message request, Authentication auth`.
- **Output:** `View` from `information`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/notes

- **Access:** Primary/collaborating TECHNICIAN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Note request, Authentication auth`.
- **Output:** `View` from `note`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/priority-override

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody PriorityOverride request,Authentication auth`.
- **Output:** `View` from `priorityOverride`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/reclassify

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody AdminCorrection request, Authentication auth`.
- **Output:** `View` from `reclassify`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/reply

- **Access:** Reporting STUDENT.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Message request, Authentication auth`.
- **Output:** `View` from `reply`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/resolve

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Resolution request, Authentication auth`.
- **Output:** `View` from `resolve`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/resume

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Message request,Authentication auth`.
- **Output:** `View` from `resume`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/start

- **Access:** Primary TECHNICIAN.
- **Inputs:** `@PathVariable Long id, Authentication auth`.
- **Output:** `View` from `start`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### POST /api/incidents/{id}/verify

- **Access:** Reporting STUDENT.
- **Inputs:** `@PathVariable Long id, @Valid @RequestBody Verification request, Authentication auth`.
- **Output:** `View` from `verify`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/incidents/my

- **Access:** STUDENT reporter.
- **Inputs:** `Authentication auth, jakarta.servlet.http.HttpServletRequest request, @RequestParam(required = false) String search, @RequestParam(required = false) IncidentStatus status, @RequestParam(required = false) Incident.Category category, @RequestParam(required = false) Incident.Priority priority`.
- **Output:** `List<View>` from `list`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

#### GET /api/incidents/options

- **Access:** Authenticated enabled account; own recipient/account scope.
- **Inputs:** `Authentication auth`.
- **Output:** `Options` from `options`.
- **Failures:** Ownership/role denial; invalid fields or lifecycle state; missing incident/configuration.

### ManualEscalationController

Source: [ManualEscalationController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ManualEscalationController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/incidents/{id}/escalation

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,Authentication auth`.
- **Output:** `Map<String,Object>` from `state`.
- **Failures:** Missing incident; mutation requires assignable incident and a higher level with reason.

#### POST /api/admin/incidents/{id}/escalation

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Request request,Authentication auth`.
- **Output:** `Map<String,Object>` from `escalate`.
- **Failures:** Missing incident; mutation requires assignable incident and a higher level with reason.

### ProblemTemplateController

Source: [ProblemTemplateController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ProblemTemplateController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/problem-templates

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<Map<String,Object>>` from `all`.
- **Failures:** Common authentication, validation and lookup failures above.

#### POST /api/admin/problem-templates

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody Template t, Authentication auth`.
- **Output:** `Map<String,Long>` from `add`.
- **Failures:** Common authentication, validation and lookup failures above.

#### PUT /api/admin/problem-templates/{id}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Long id,@Valid @RequestBody Template t,Authentication auth`.
- **Output:** `Map<String,Long>` from `edit`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/student/problem-templates

- **Access:** STUDENT; own scope.
- **Inputs:** None.
- **Output:** `List<Map<String,Object>>` from `available`.
- **Failures:** Common authentication, validation and lookup failures above.

### SlaController

Source: [SlaController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/SlaController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/sla

- **Access:** ADMIN.
- **Inputs:** `Authentication auth`.
- **Output:** `List<View>` from `list`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/admin/sla-policies

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `List<SlaPolicy>` from `policies`.
- **Failures:** Targets outside 1–525600 minutes, resolution shorter than response, missing priority policy.

#### PUT /api/admin/sla-policies/{priority}

- **Access:** ADMIN.
- **Inputs:** `@PathVariable Incident.Priority priority, @Valid @RequestBody Targets targets`.
- **Output:** `SlaPolicy` from `update`.
- **Failures:** Targets outside 1–525600 minutes, resolution shorter than response, missing priority policy.

#### GET /api/student/sla

- **Access:** STUDENT; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<View>` from `list`.
- **Failures:** Common authentication, validation and lookup failures above.

#### GET /api/technician/sla

- **Access:** TECHNICIAN; own scope.
- **Inputs:** `Authentication auth`.
- **Output:** `List<View>` from `list`.
- **Failures:** Common authentication, validation and lookup failures above.

### PriorityController

Source: [PriorityController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/priority/PriorityController.java). Signature names refer to DTO records shown in the contract section below or linked implementation. Authentication and servlet objects are injected server-side, not client JSON.

#### GET /api/admin/priority-strategy

- **Access:** ADMIN.
- **Inputs:** None.
- **Output:** `PrioritySettings` from `current`.
- **Failures:** Invalid weights/thresholds/matrix/rules or unavailable service criticality.

#### PUT /api/admin/priority-strategy

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody Configuration request, Authentication auth`.
- **Output:** `PrioritySettings` from `update`.
- **Failures:** Invalid weights/thresholds/matrix/rules or unavailable service criticality.

#### POST /api/admin/priority-strategy/preview

- **Access:** ADMIN.
- **Inputs:** `@Valid @RequestBody Preview request`.
- **Output:** `PreviewResult` from `preview`.
- **Failures:** Invalid weights/thresholds/matrix/rules or unavailable service criticality.

### Session logout (security filter)

POST /api/auth/logout is implemented by SecurityConfig rather than a controller. Send the session cookie and CSRF token; no body. It clears the session cookie and returns HTTP 200 with a signed-out message.

## DTO contract excerpts

These declarations are extracted directly from the source. Nested response/value types resolve in the linked files. Bean annotations express required fields/lengths; service methods additionally validate eligibility, uniqueness, catalog settings and lifecycle state.

### IncidentData records

[IncidentData.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/IncidentData.java)

```java
public record Create(@NotBlank @Size(max = 100) String title, @NotBlank @Size(min = 20, max = 2000) String description,
            @NotNull Category category, ItService affectedService, Level impact, Level urgency, @Min(1) Integer affectedUsers)

public record Classification(@NotNull Category category)

public record AdminCorrection(@NotNull ItService affectedService, Category category, @NotNull Level impact, @NotNull Level urgency,
            @NotBlank @Size(min = 5, max = 500) String reason)

public record Edit(@NotBlank @Size(max = 100) String title, @NotBlank @Size(min = 20, max = 2000) String description)

public record Assignment(@NotNull @Positive Long technicianId, @Size(max=2000) String reason)

public record Note(@NotBlank @Size(max = 2000) String content, @NotNull Visibility visibility)

public record Message(@NotBlank @Size(max = 2000) String content)

public record InternalMessage(@NotBlank @Size(max = 2000) String content, boolean urgent)

public record PriorityOverride(@NotNull Priority priority,@NotBlank @Size(min=5,max=2000) String reason)

public record Disposition(@NotNull ClosureKind kind, @NotBlank @Size(min=5,max=2000) String reason, @Positive Long duplicateOf)

public record Resolution(@NotBlank @Size(max = 2000) String summary, @NotBlank @Size(max = 2000) String steps)

public record Verification(@NotNull Boolean confirmed, @Size(max = 2000) String reason)

public record Feedback(@NotNull @Min(1) @Max(5) Integer rating, @Size(max = 1000) String comment)

public record Person(Long id, String name)

public record TeamMember(Person technician, String responsibility)

public record NoteView(Long id, Person author, String content, Visibility visibility, InvestigationNote.Kind kind, Instant createdAt)

public record HistoryView(Long id, Person actor, String action, String description, Instant createdAt)

public record FeedbackView(int rating, String comment, Instant createdAt)

public record View(Long id, String title, String description, Person reporter, Category category, ItService affectedService,
            String serviceLabel, Level impact, Level urgency, Priority priority, IncidentStatus status, Person assignedTechnician,
            Instant createdAt, Instant updatedAt, Instant acknowledgedAt, Instant resolvedAt, Instant verifiedAt, Instant closedAt,
            String resolutionSummary, String resolutionSteps, List<NoteView> notes, List<HistoryView> history, FeedbackView feedback, boolean primaryOwner, List<TeamMember> collaborators, com.bits.itsm.intelligence.CategorySuggester.Suggestion categorySuggestion, Level criticality, Instant responseDeadline, Instant resolutionDeadline, SlaClock.Status responseSla, SlaClock.Status resolutionSla,
            boolean needsClassificationReview, boolean autoClassified, String classificationNotes, Category reportedCategory, Level reportedUrgency, boolean awaitingStudent)

public record Option(String value, String label)

public record Options(List<Option> categories, List<Option> services)
```

### AssignmentData records

[AssignmentData.java](../bits-itsm/backend/src/main/java/com/bits/itsm/assignment/AssignmentData.java)

```java
public record Strategy(@NotNull AssignmentSettings.Mode strategy)

public record AvailabilityUpdate(@NotNull Technician.Availability availability)

public record Profile(@NotEmpty Set<@NotNull TechnicianSkill> skills, @NotNull Technician.Availability availability, @Min(1) @Max(100) int maxWorkload)

public record Transfer(@Positive Long technicianId, @NotNull TransferRequest.Reason reason, @NotBlank @Size(min = 10, max = 2000) String handoverNotes)

public record Collaboration(@NotNull @Positive Long technicianId, @NotNull TechnicianSkill requiredSkill,
            @NotBlank @Size(max = 2000) String reason, @NotBlank @Size(max = 2000) String responsibility)

public record Review(@NotBlank @Size(max = 2000) String notes, @Positive Long technicianId)

public record RequestView(Long id, Long incidentId, String incidentTitle, Person primaryTechnician, Person proposedTechnician, Person assignedTechnician,
            AssignmentRequest.Status status, String reason, String handoverNotes, TechnicianSkill requiredSkill, String responsibility,
            Instant requestedAt, Person reviewedBy, Instant reviewedAt, String reviewNotes)
```

### AdminManagementController records

[AdminManagementController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/api/AdminManagementController.java)

```java
public record UserView(Long id, String name, String email, Role role, boolean enabled, String studentNumber, Technician.Availability availability)

public record NewUser(@NotBlank @Size(max=100) String name, @NotBlank String email, @NotNull Role role,
            @NotBlank @Size(min=12,max=72) String password, @Size(max=30) String studentNumber, TechnicianSkill skill)

public record UserEdit(@NotBlank @Size(max=100) String name, boolean enabled)

public record CatalogEdit(@NotBlank @Size(max=100) String label, boolean enabled, Incident.Level criticality, @Size(max=2000) String keywords, @Size(max=2000) String description, TechnicianSkill requiredSkill)

public record NewCategory(@NotBlank String code,@NotBlank @Size(max=100) String label,@Size(max=2000) String keywords,boolean enabled,TechnicianSkill requiredSkill)

public record NewService(@NotBlank String code, @NotBlank @Size(max=100) String label, @NotBlank @Size(max=2000) String description, @NotBlank @Size(max=2000) String keywords, Incident.Level criticality, boolean enabled)
```

### PriorityController records

[PriorityController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/priority/PriorityController.java)

```java
public record Configuration(@NotNull PrioritySettings.Mode strategy, double impactWeight, double urgencyWeight, double criticalityWeight,
            PriorityStrategy.Thresholds multiplicativeThresholds, PriorityStrategy.Thresholds weightedThresholds,
            List<Priority> matrix, Integer criticalBoost, List<RuleBasedPriorityStrategy.Rule> rules, Priority fallback)

public record Preview(@NotNull @Valid Configuration configuration, @NotNull Level impact, @NotNull Level urgency, @NotNull ItService affectedService)

public record PreviewResult(Priority priority, Level criticality)
```

### PasswordController records

[PasswordController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/auth/PasswordController.java)

```java
public record Input(String token,String currentPassword,String password,String confirmation)
```

### AuthController records

[AuthController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/auth/AuthController.java)

```java
public record Registration(String name, String email, String password, String confirmPassword)

public record LoginRequest(@NotBlank @Size(max = 254) String email,
            @NotBlank @Size(max = 72) String password, Role role)

public record Account(Long id, String name, String email, Role role)
```

### ProblemTemplateController records

[ProblemTemplateController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ProblemTemplateController.java)

```java
public record Template(@NotBlank @Size(max=100) String title, @NotBlank @Size(max=100) String category,
            @NotNull @Size(max=2000) String description, boolean enabled, @Min(0) int displayOrder,
            @NotNull @Size(max=2000) String keywords, String serviceCode, String categoryCode)
```

### SlaController records

[SlaController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/SlaController.java)

```java
public record Targets(@Min(1) @Max(525600) int responseMinutes, @Min(1) @Max(525600) int resolutionMinutes)

public record View(Long incidentId, String title, Instant responseDeadline, Instant resolutionDeadline, SlaClock.Status responseStatus, SlaClock.Status resolutionStatus)
```

### ManualEscalationController records

[ManualEscalationController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ManualEscalationController.java)

```java
public record Request(@NotNull EscalationLevel level,@NotBlank @Size(min=5,max=2000) String reason)
```

### ConversationController records

[ConversationController.java](../bits-itsm/backend/src/main/java/com/bits/itsm/incident/ConversationController.java)

```java
public record Read(@NotNull Long messageId)
```

Inventory: **100 expanded controller routes**, plus the security-filter logout route. Frontend SPA forwards are not JSON APIs. The earlier 97-mapping inventory counted each array annotation once and omitted aliases; this count reflects documentation completeness, not new endpoints.
