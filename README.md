import streamlit as st
import pandas as pd
import os

# ============================================================
# STUDENT GRADE MANAGEMENT SYSTEM
# Framework: Streamlit
# ============================================================

DATA_FILE = "students.csv"

# ------------------------------------------------------------
# Page Configuration
# ------------------------------------------------------------

st.set_page_config(
    page_title="Student Grade System",
    page_icon="🎓",
    layout="wide"
)

# ------------------------------------------------------------
# Grade Calculation
# ------------------------------------------------------------

def calculate_grade(percentage):
    if percentage >= 90:
        return "A+"
    elif percentage >= 80:
        return "A"
    elif percentage >= 70:
        return "B"
    elif percentage >= 60:
        return "C"
    elif percentage >= 50:
        return "D"
    elif percentage >= 40:
        return "E"
    else:
        return "F"


def calculate_result(marks):
    if all(mark >= 35 for mark in marks):
        return "PASS"
    else:
        return "FAIL"


# ------------------------------------------------------------
# Load Existing Data
# ------------------------------------------------------------

def load_data():
    if os.path.exists(DATA_FILE):
        return pd.read_csv(DATA_FILE)

    return pd.DataFrame(
        columns=[
            "Student ID",
            "Student Name",
            "Class",
            "Section",
            "Maths",
            "Physics",
            "Chemistry",
            "English",
            "Computer Science",
            "Total",
            "Percentage",
            "Grade",
            "Result"
        ]
    )


# ------------------------------------------------------------
# Save Data
# ------------------------------------------------------------

def save_data(data):
    data.to_csv(DATA_FILE, index=False)


# Load database
df = load_data()


# ============================================================
# SIDEBAR
# ============================================================

st.sidebar.title("🎓 Student Grade System")

menu = st.sidebar.radio(
    "Select Option",
    [
        "🏠 Dashboard",
        "➕ Add Student",
        "📋 View Students",
        "🔎 Search Student",
        "📊 Student Report",
        "🗑️ Delete Student"
    ]
)


# ============================================================
# DASHBOARD
# ============================================================

if menu == "🏠 Dashboard":

    st.title("🎓 Student Grade Management System")

    st.write(
        "Interactive student performance management system "
        "built using Python and Streamlit."
    )

    st.divider()

    # Statistics
    total_students = len(df)

    if total_students > 0:

        passed_students = len(
            df[df["Result"] == "PASS"]
        )

        failed_students = len(
            df[df["Result"] == "FAIL"]
        )

        average_percentage = df["Percentage"].mean()

        highest_percentage = df["Percentage"].max()

        col1, col2, col3, col4 = st.columns(4)

        col1.metric(
            "👨‍🎓 Total Students",
            total_students
        )

        col2.metric(
            "✅ Passed",
            passed_students
        )

        col3.metric(
            "❌ Failed",
            failed_students
        )

        col4.metric(
            "📈 Average %",
            f"{average_percentage:.2f}%"
        )

        st.divider()

        # Top students
        st.subheader("🏆 Top Performing Students")

        top_students = df.sort_values(
            by="Percentage",
            ascending=False
        ).head(10)

        st.dataframe(
            top_students[
                [
                    "Student ID",
                    "Student Name",
                    "Class",
                    "Percentage",
                    "Grade",
                    "Result"
                ]
            ],
            use_container_width=True
        )

        st.divider()

        # Grade distribution
        st.subheader("📊 Grade Distribution")

        grade_counts = df["Grade"].value_counts()

        st.bar_chart(grade_counts)

    else:

        st.info(
            "No student records available. "
            "Please add a student using the 'Add Student' option."
        )


# ============================================================
# ADD STUDENT
# ============================================================

elif menu == "➕ Add Student":

    st.title("➕ Add Student")

    st.write("Enter the student's details and marks.")

    st.divider()

    with st.form("student_form"):

        col1, col2 = st.columns(2)

        with col1:

            student_id = st.text_input(
                "Student ID",
                placeholder="Example: ST001"
            )

            student_name = st.text_input(
                "Student Name",
                placeholder="Enter student name"
            )

            student_class = st.selectbox(
                "Class",
                [
                    "10",
                    "11",
                    "12"
                ]
            )

        with col2:

            section = st.selectbox(
                "Section",
                [
                    "A",
                    "B",
                    "C",
                    "D"
                ]
            )

            st.write("### Subject Marks")

            maths = st.number_input(
                "Mathematics",
                min_value=0,
                max_value=100,
                value=0
            )

            physics = st.number_input(
                "Physics",
                min_value=0,
                max_value=100,
                value=0
            )

            chemistry = st.number_input(
                "Chemistry",
                min_value=0,
                max_value=100,
                value=0
            )

            english = st.number_input(
                "English",
                min_value=0,
                max_value=100,
                value=0
            )

            computer_science = st.number_input(
                "Computer Science",
                min_value=0,
                max_value=100,
                value=0
            )

        submitted = st.form_submit_button(
            "💾 Save Student",
            use_container_width=True
        )

        if submitted:

            # Validation

            if not student_id.strip():

                st.error("Please enter Student ID.")

            elif not student_name.strip():

                st.error("Please enter Student Name.")

            elif student_id in df["Student ID"].astype(str).values:

                st.error(
                    "Student ID already exists."
                )

            else:

                # Calculate marks

                marks = [
                    maths,
                    physics,
                    chemistry,
                    english,
                    computer_science
                ]

                total = sum(marks)

                percentage = total / 5

                grade = calculate_grade(
                    percentage
                )

                result = calculate_result(
                    marks
                )

                # Create new student

                new_student = {
                    "Student ID": student_id,
                    "Student Name": student_name,
                    "Class": student_class,
                    "Section": section,
                    "Maths": maths,
                    "Physics": physics,
                    "Chemistry": chemistry,
                    "English": english,
                    "Computer Science": computer_science,
                    "Total": total,
                    "Percentage": percentage,
                    "Grade": grade,
                    "Result": result
                }

                # Add to dataframe

                df = pd.concat(
                    [
                        df,
                        pd.DataFrame([new_student])
                    ],
                    ignore_index=True
                )

                # Save database

                save_data(df)

                st.success(
                    f"Student {student_name} added successfully!"
                )

                # Display result

                st.subheader("📋 Student Result")

                col1, col2, col3 = st.columns(3)

                col1.metric(
                    "Total Marks",
                    f"{total}/500"
                )

                col2.metric(
                    "Percentage",
                    f"{percentage:.2f}%"
                )

                col3.metric(
                    "Grade",
                    grade
                )

                if result == "PASS":
                    st.success(
                        "🎉 Student has PASSED."
                    )
                else:
                    st.error(
                        "Student has FAILED."
                    )


# ============================================================
# VIEW STUDENTS
# ============================================================

elif menu == "📋 View Students":

    st.title("📋 All Students")

    if len(df) == 0:

        st.info("No student records found.")

    else:

        st.dataframe(
            df,
            use_container_width=True,
            hide_index=True
        )

        st.divider()

        st.subheader("📥 Download Student Data")

        csv_data = df.to_csv(
            index=False
        ).encode("utf-8")

        st.download_button(
            label="⬇️ Download CSV",
            data=csv_data,
            file_name="student_results.csv",
            mime="text/csv"
        )


# ============================================================
# SEARCH STUDENT
# ============================================================

elif menu == "🔎 Search Student":

    st.title("🔎 Search Student")

    search_text = st.text_input(
        "Enter Student ID or Name"
    )

    if search_text:

        result = df[
            df["Student ID"]
            .astype(str)
            .str.contains(
                search_text,
                case=False,
                na=False
            )
            |
            df["Student Name"]
            .astype(str)
            .str.contains(
                search_text,
                case=False,
                na=False
            )
        ]

        if len(result) > 0:

            st.success(
                f"{len(result)} student(s) found."
            )

            st.dataframe(
                result,
                use_container_width=True,
                hide_index=True
            )

        else:

            st.warning(
                "No student found."
            )


# ============================================================
# STUDENT REPORT
# ============================================================

elif menu == "📊 Student Report":

    st.title("📊 Student Report")

    if len(df) == 0:

        st.info("No student records available.")

    else:

        student_list = df[
            "Student Name"
        ].tolist()

        selected_student = st.selectbox(
            "Select Student",
            student_list
        )

        student = df[
            df["Student Name"]
            == selected_student
        ].iloc[0]

        st.divider()

        # Student information

        st.subheader("👨‍🎓 Student Information")

        col1, col2, col3, col4 = st.columns(4)

        col1.write("**Student ID**")
        col1.write(student["Student ID"])

        col2.write("**Student Name**")
        col2.write(student["Student Name"])

        col3.write("**Class**")
        col3.write(student["Class"])

        col4.write("**Section**")
        col4.write(student["Section"])

        st.divider()

        # Marks

        st.subheader("📚 Subject Marks")

        marks_data = pd.DataFrame(
            {
                "Subject": [
                    "Mathematics",
                    "Physics",
                    "Chemistry",
                    "English",
                    "Computer Science"
                ],
                "Marks": [
                    student["Maths"],
                    student["Physics"],
                    student["Chemistry"],
                    student["English"],
                    student["Computer Science"]
                ]
            }
        )

        st.dataframe(
            marks_data,
            use_container_width=True,
            hide_index=True
        )

        st.divider()

        # Result

        st.subheader("🏆 Final Result")

        col1, col2, col3, col4 = st.columns(4)

        col1.metric(
            "Total",
            f"{student['Total']}/500"
        )

        col2.metric(
            "Percentage",
            f"{student['Percentage']:.2f}%"
        )

        col3.metric(
            "Grade",
            student["Grade"]
        )

        col4.metric(
            "Result",
            student["Result"]
        )

        st.divider()

        # Performance chart

        st.subheader("📈 Performance Analysis")

        chart_data = pd.DataFrame(
            {
                "Marks": [
                    student["Maths"],
                    student["Physics"],
                    student["Chemistry"],
                    student["English"],
                    student["Computer Science"]
                ]
            },
            index=[
                "Mathematics",
                "Physics",
                "Chemistry",
                "English",
                "Computer Science"
            ]
        )

        st.bar_chart(chart_data)


# ============================================================
# DELETE STUDENT
# ============================================================

elif menu == "🗑️ Delete Student":

    st.title("🗑️ Delete Student")

    if len(df) == 0:

        st.info("No student records available.")

    else:

        student_ids = df[
            "Student ID"
        ].tolist()

        selected_id = st.selectbox(
            "Select Student ID",
            student_ids
        )

        student = df[
            df["Student ID"]
            == selected_id
        ].iloc[0]

        st.warning(
            f"You are about to delete "
            f"**{student['Student Name']}** "
            f"({selected_id})."
        )

        confirm = st.checkbox(
            "I confirm that I want to delete this student."
        )

        if st.button(
            "🗑️ Delete Student",
            type="primary"
        ):

            if confirm:

                df = df[
                    df["Student ID"]
                    != selected_id
                ]

                save_data(df)

                st.success(
                    "Student deleted successfully."
                )

                st.rerun()

            else:

                st.error(
                    "Please confirm deletion first."
                )


# ============================================================
# FOOTER
# ============================================================

st.sidebar.divider()

st.sidebar.info(
    "🎓 Student Grade Management System\n\n"
    "Built with Python + Streamlit + Pandas"
)
