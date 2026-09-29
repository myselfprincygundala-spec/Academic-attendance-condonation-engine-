# Academic-attendance-condonation-engine-
This production-ready system tracks student attendance metrics across multiple courses, flags students dropping below the institutional mandatory 75% minimum threshold, distinguishes between students who qualify for a medical/justified exception vs. those who must pay a condonation fee.
from datetime import datetime
import json
import random
from typing import Dict, List, Any, Tuple, Optional

# =====================================================================
# 1. STRUCTURAL DATA CONFIGURATION & POLICY CONSTANTS
# =====================================================================

class UniversityPolicy:
    """Defines strict institutional rules regarding attendance boundaries and fines."""
    MANDATORY_THRESHOLD = 75.0      # 75% attendance required for regular clearance
    CONDONATION_MINIMUM = 65.0      # Below 65% is generally a hard bar from exams
    CONDONATION_FEE_AMOUNT = 1500.00 # Condonation processing fine in currency units
    MEDICAL_EXEMPTION_ALLOWED = True # Allows skipping fine if medical certificates exist


# =====================================================================
# 2. CORE DOMAIN OBJECT: STUDENT ATTENDANCE RECORD
# =====================================================================

class StudentAttendanceProfile:
    """Manages tracking records, absence classification, and fine calculations."""
    def __init__(self, student_id: str, name: str, academic_program: str):
        self.student_id = student_id
        self.name = name
        self.academic_program = academic_program
        
        # Course registry mapping: {"Course_Code": (total_sessions, attended_sessions)}
        self.course_ledger: Dict[str, Tuple[int, int]] = {}
        self.has_certified_medical_excuse = False

    def register_course_data(self, course_code: str, total_sessions: int, attended_sessions: int):
        """Safely logs structural session logs validation checking parameters."""
        if attended_sessions > total_sessions:
            raise ValueError(f"Attended sessions ({attended_sessions}) cannot exceed total classes ({total_sessions}).")
        if total_sessions <= 0:
            raise ValueError("Total structural classes must be greater than zero.")
        
        self.course_ledger[course_code] = (total_sessions, attended_sessions)

    def submit_medical_verification(self, verified: bool):
        """Logs if official health documentation was processed by the dean's administration."""
        self.has_certified_medical_excuse = verified

    def evaluate_course_eligibility(self, course_code: str) -> Dict[str, Any]:
        """Runs compliance algorithms to cross-reference performance metrics against policies."""
        record = self.course_ledger.get(course_code)
        if not record:
            return {"status": "Unregistered"}

        total, attended = record
        attendance_percentage = round((attended / total) * 100, 2)
        
        # Base Evaluation Matrix Indicators
        status = "COMPLIANT"
        fee_applicable = 0.0
        action_required = "None. Cleared for terminal examinations."
        
        if attendance_percentage >= UniversityPolicy.MANDATORY_THRESHOLD:
            status = "COMPLIANT"
        elif attendance_percentage >= UniversityPolicy.CONDONATION_MINIMUM:
            if self.has_certified_medical_excuse:
                status = "CONDONED_MEDICAL"
                fee_applicable = 0.0
                action_required = "Medical waiver processed. Waived fine. Clear for exams."
            else:
                status = "CONDONATION_REQUIRED"
                fee_applicable = UniversityPolicy.CONDONATION_FEE_AMOUNT
                action_required = f"Must clear a condonation fee of ${fee_applicable:.2f} to sit for exams."
        else:
            status = "DETAINED_BARRED"
            action_required = "Attendance critically low. Strictly barred from terminal examinations."

        return {
            "total_classes": total,
            "attended_classes": attended,
            "attendance_percentage": attendance_percentage,
            "compliance_status": status,
            "financial_penalty": fee_applicable,
            "administrative_action": action_required
        }


# =====================================================================
# 3. CONTROL CENTER: REGISTRAR COMPLIANCE DASHBOARD
# =====================================================================

class AttendanceRegistrarConsole:
    """Orchestrates university-wide auditing, fine collection billing, and metrics generation."""
    def __init__(self, semester_id: str):
        self.semester_id = semester_id
        self.student_roster: Dict[str, StudentAttendanceProfile] = {}

    def enroll_student(self, student: StudentAttendanceProfile):
        self.student_roster[student.student_id] = student

    def generate_compliance_audit_report(self) -> Dict[str, Any]:
        """Compiles an intensive data blueprint detailing academic eligibility and penalty pipelines."""
        audit_log = {
            "audit_timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "semester_scope": self.semester_id,
            "policy_benchmarks": {
                "regular_clearance_pct": UniversityPolicy.MANDATORY_THRESHOLD,
                "minimum_condonable_pct": UniversityPolicy.CONDONATION_MINIMUM,
                "standard_fine_amount": UniversityPolicy.CONDONATION_FEE_AMOUNT
            },
            "roster_summaries": [],
            "financial_billing_queue": [],
            "examination_bar_list": []
        }

        total_collected_fines = 0.0

        for s_id, student in self.student_roster.items():
            profile_manifest = {
                "student_id": s_id,
                "name": student.name,
                "program": student.academic_program,
                "medical_status_logged": student.has_certified_medical_excuse,
                "course_audits": {}
            }
            
            is_flagged_for_fines = False
            student_total_fine = 0.0

            for course_code in student.course_ledger.keys():
                course_audit = student.evaluate_course_eligibility(course_code)
                profile_manifest["course_audits"][course_code] = course_audit
                
                # Aggregate financial implications
                if course_audit["compliance_status"] == "CONDONATION_REQUIRED":
                    is_flagged_for_fines = True
                    student_total_fine += course_audit["financial_penalty"]
                
                # Segregate extreme terminal failures
                if course_audit["compliance_status"] == "DETAINED_BARRED":
                    audit_log["examination_bar_list"].append({
                        "student_id": s_id,
                        "name": student.name,
                        "course": course_code,
                        "recorded_percentage": f"{course_audit['attendance_percentage']}%"
                    })

            audit_log["roster_summaries"].append(profile_manifest)

            # Route to billing manifest if fine constraints triggered
            if is_flagged_for_fines:
                total_collected_fines += student_total_fine
                audit_log["financial_billing_queue"].append({
                    "student_id": s_id,
                    "name": student.name,
                    "aggregated_fine_due": student_total_fine,
                    "payment_status": "PENDING_REGISTRAR_CLEARANCE"
                })

        audit_log["expected_total_condonation_revenue"] = total_collected_fines
        return audit_log


# =====================================================================
# 4. RUNTIME SYSTEM INTENT SIMULATION
# =====================================================================

if __name__ == "__main__":
    # Create registrar container loop for the current Academic Semester
    registrar_office = AttendanceRegistrarConsole(semester_id="FALL_2026_V1")

    # --- Student 1: Perfect Compliance Profile ---
    stu_1 = StudentAttendanceProfile(student_id="STU_2026_01", name="Arjun Varma", academic_program="B.Tech Computer Science")
    stu_1.register_course_data(course_code="CS-101", total_sessions=60, attended_sessions=55) # 91.67%
    stu_1.register_course_data(course_code="CS-102", total_sessions=45, attended_sessions=40) # 88.89%
    registrar_office.enroll_student(stu_1)

    # --- Student 2: Condonation Fee Mandatory Profile (Attendance between 65% - 75%) ---
    stu_2 = StudentAttendanceProfile(student_id="STU_2026_02", name="Deepika Rao", academic_program="B.Tech Data Science")
    stu_2.register_course_data(course_code="CS-101", total_sessions=60, attended_sessions=43) # 71.67% -> Condonation Needed!
    stu_2.register_course_data(course_code="CS-102", total_sessions=45, attended_sessions=38) # 84.44% -> Clear
    registrar_office.enroll_student(stu_2)

    # --- Student 3: Condonable Grade with Verified Medical Waiver ---
    stu_3 = StudentAttendanceProfile(student_id="STU_2026_03", name="Karan Malhotra", academic_program="B.Tech CyberSecurity")
    stu_3.register_course_data(course_code="CS-101", total_sessions=60, attended_sessions=41) # 68.33% -> Condonable Boundary
    stu_3.register_course_data(course_code="CS-102", total_sessions=45, attended_sessions=42) # 93.33% -> Clear
    stu_3.submit_medical_verification(verified=True) # Exemption Flag Submitted!
    registrar_office.enroll_student(stu_3)

    # --- Student 4: Critically Low Profile (Below 65% - Detained/Barred) ---
    stu_4 = StudentAttendanceProfile(student_id="STU_2026_04", name="Siddharth Sen", academic_program="B.Tech AI & ML")
    stu_4.register_course_data(course_code="CS-101", total_sessions=60, attended_sessions=32) # 53.33% -> Barred completely!
    stu_4.register_course_data(course_code="CS-102", total_sessions=45, attended_sessions=20) # 44.44% -> Barred completely!
    registrar_office.enroll_student(stu_4)

    # Execute entire pipeline automation infrastructure
    complete_university_audit_sheet = registrar_office.generate_compliance_audit_report()

    print("==================================================================")
