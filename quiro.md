``` mermaid
erDiagram
	OperatingRooms {
		int ORID PK "Unique identifier for each operating room."
		string Name "Name or number of the operating room."
		string Location "Hospital or facility location."
		string Status "Availability status (Available, In Use, Maintenance)."
		string Type "Type of OR based on the procedures (General, Cardiac, Orthopedic, etc.)."
	}
	Surgeries {
		int SurgeryID PK "Unique identifier for each surgery in AgendaQuirofano"
		int QFSurgeryID "Unique SurgeryID from QuirófanoFácil"
		int ORID FK "Linked to OperatingRooms table"
		int ProcedureID FK "Linked to Procedures table"
		int PatientID FK "Linked to Patients table"
		datetime ScheduledStart "Scheduled start time of the surgery"
		datetime ScheduledEnd "Scheduled end time of the surgery"
		datetime ActualStart "Actual start time"
		datetime ActualEnd "Actual end time"
		string Status "Current status (Scheduled, In Progress, Completed, Cancelled)"
	}
	Procedures {
		int ProcedureID PK "Unique identifier for each procedure."
		string Name "Name of the procedure."
		string Description "Brief description of the procedure."
		int Duration "Expected duration in minutes."
		string RequiredResources "List or JSON object detailing the required resources (equipment, personnel)."
	}
	Staff {
		int StaffID PK "Unique identifier for each staff member."
		string Name "Name of the staff member."
		string Role "Role within the hospital (Surgeon, Nurse, Anesthesiologist, etc.)."
		string Specialization "Medical or technical specialization."
	}
	Resources {
		int ResourceID PK "Unique identifier for each resource."
		string Name "Name of the resource (equipment, tool)."
		string Type "Type of resource (Medical Equipment, Surgical Instrument, etc.)."
		string AvailabilityStatus "Current availability status."
	}
	EfficiencyMetrics {
		int MetricID PK "Unique identifier for each metric."
		int ORID FK "Linked to OperatingRooms table."
		date Date "Date of the metric record."
		float UtilizationRate "Percentage of time the OR was in use."
		int TurnoverTime "Time taken to prepare the OR for the next procedure in minutes."
		int ProcedureCount "Number of procedures performed."
	}
	Roles {
		int RoleID PK "Unique identifier for each role."
		string RoleName "Name of the role (Admin, Surgeon, Scheduler, etc.)."
		string Description "Description of the role's permissions and access."
	}
	ModuleSubscriptions {
		int SubscriptionID PK "Unique identifier for each subscription."
		int RoleID FK "Linked to Roles table."
		string ModuleName "Name of the module (Scheduling, Resource Management, etc.)."
		string AccessLevel "Type of access (Read, Write, Edit, Delete)."
	}
	ActionPermissions {
		int PermissionID PK "Unique identifier for each permission."
		int RoleID FK "Linked to Roles table."
		string ActionName "Name of the action (Create Schedule, Allocate Resource, View Metrics)."
		bool Allowed "Boolean indicating if the action is allowed for the role."
	}
	ReservationRequests {
		int RequestID PK "Unique identifier for each reservation request."
		int ORID FK "Linked to OperatingRooms table, indicating the requested OR."
		int RequestedBy FK "ID linking to the Staff table for the person who submitted the request."
		datetime BlockStart "Proposed start time of the procedure."
		datetime BlockEnd "Proposed end time of the procedure."
		string Status "Current status (Submitted, Under Review, Approved, Rejected, etc.)"
		datetime DateSubmitted "Timestamp when the request was submitted."
		string RequiredResources "List or JSON object detailing the required resources (equipment, personnel)."
	}
	RequestedProcedures {
		int RequestedProcedureID PK "Unique requested procedure identifier."
		int RequestID FK "Linked to the ReservationRequests table."
		int ProcedureID FK "Procedure ID from the Procedures table."
		int EstimatedDuration "Estimated duration of the procedure."
		string Priority "Priority of the procedure within the block (to handle sequencing or importance)."
	}
	RequestHandlers {
		int HandlerID PK "Unique identifier for each potential approver/handler."
		int RequestID FK "Linked to ReservationRequests table."
		int HandlerRoleID FK "Linked to Roles table, indicating the role of the handler."
		int Order "Order in the chain of responsibility.."
		string Status "Status of the handler's review (Pending, Approved, Rejected, Passed)."
	}
	RequestStateTransitions {
		int TransitionID PK "Unique identifier for each approval action."
		int RequestID FK "Linked to ReservationRequests table."
		int PreviousState FK "Previous state of the request."
		int NewState FK "New state after the transition."
		datetime Timestamp "Time when the transition occurred."
	}
	Observers {
		int ObserverID PK "Unique identifier for each observer."
		int StaffID FK "Linked to Staff table, indicating the observer."
		int ObserverRoleID FK "Linked to Roles table."
		string Interest "Type of updates the observer is interested in (e.g., All, Only Approvals, Only Rejections)."
		bool Notify "Boolean indicating if the observer should be notified on updates."
	}
	Notifications {
		int NotificationID PK "Unique identifier for each notification."
		int RequestID FK "Linked to ReservationRequests table."
		int ObserverID FK "Linked to Observers table."
		string Message "Notification message detailing the update."
		datetime Timestamp "When the notification was sent."
	}
	Commands {
		int CommandID PK "Unique identifier for each command/action."
		string Name "Name of the the decision made (Approve, Reject, Pass)."
		string Description "Description of what the command does."
	}
	RequestCommands {
		int RequestCommandID PK "Unique identifier for each request command instance."
		int RequestID FK "Linked to ReservationRequests table."
		int CommandID FK "Linked to Commands table."
		int ExecutedBy FK "ID linking to Staff table, indicating who executed the command."
		datetime Timestamp "When the command was executed."
		string Outcome "Result of the command execution (e.g., Success, Failed, Reverted)."
		string Comments "Any comments or reasons provided by the handler."
	}

	Commands ||--o{ RequestCommands : "triggers"
	Observers ||--o{ Notifications : "receives"
	OperatingRooms ||--o{ Surgeries : "hosts"
	OperatingRooms ||--o{ EfficiencyMetrics : "measured in"
	OperatingRooms ||--o{ ReservationRequests : "requested"
	Procedures ||--o{ RequestedProcedures : "lists"
	RequestedProcedures ||--o{ ReservationRequests : "performs"
	ReservationRequests ||--o{ Notifications : "notifies"
	ReservationRequests ||--o{ Commands : "undergoes"
	ReservationRequests ||--o{ RequestHandlers : "handled by"
	ReservationRequests ||--o{ Resources : "may require"
	RequestStateTransitions ||--o{ ReservationRequests : "audits"
	Roles ||--o{ ModuleSubscriptions : "subscribes to"
	Roles ||--o{ ActionPermissions : "grants"
	Roles ||--o{ RequestHandlers : "assigns"
	Roles ||--o{ Observers : "assigns"
	Surgeries ||--o{ Procedures : "implement"
	Staff ||--o{ Observers : "observed by"
	Staff ||--o{ ReservationRequests : "requests"
	Staff ||--o{ RequestCommands : "executes"	
```
