#include <iostream>
#include <vector>
#include <string>
#include <fstream>

class Student {
public:
    std::string indexNumber;
    std::string name;

    Student(std::string idx, std::string n) : indexNumber(idx), name(n) {}
};

class AttendanceSession {
public:
    std::string courseCode;
    std::string date;
    std::vector<std::pair<Student, char>> attendance; // char: 'P' = Present, 'A' = Absent, 'L' = Late

    AttendanceSession(std::string cc, std::string d) : courseCode(cc), date(d) {}

    void markAttendance(Student s, char status) {
        attendance.push_back({s, status});
    }
};

// Saving/Loading functions
void saveStudents(const std::vector<Student>& students) {
    std::ofstream file("students.txt");
    for (const auto& s : students) {
        file << s.indexNumber << "," << s.name << "\n";
    }
    file.close();
}

std::vector<Student> loadStudents() {
    std::vector<Student> students;
    std::ifstream file("students.txt");
    if (!file.is_open()) return students; // Handle missing file
    std::string line;
    while (std::getline(file, line)) {
        size_t comma = line.find(',');
        if (comma != std::string::npos) {
            std::string idx = line.substr(0, comma);
            std::string name = line.substr(comma + 1);
            students.push_back(Student(idx, name));
        }
    }
    file.close();
    return students;
}

void saveSession(const AttendanceSession& session) {
    std::ofstream file("session_" + session.courseCode + "_" + session.date + ".txt");
    for (const auto& a : session.attendance) {
        file << a.first.indexNumber << "," << a.second << "\n";
    }
    file.close();
}

int main() {
    std::vector<Student> students = loadStudents();
    std::vector<AttendanceSession> sessions;
    int choice;

    do {
        std::cout << "Digital Attendance System\n";
        std::cout << "1. Register student\n";
        std::cout << "2. View students\n";
        std::cout << "3. Create lecture session\n";
        std::cout << "4. Mark attendance\n";
        std::cout << "5. Attendance summary\n";
        std::cout << "6. Exit\n";
        std::cout << "Enter choice: ";
        std::cin >> choice;

        switch (choice) {
        case 1: {
            std::string index, name;
            std::cout << "Enter index number and name: ";
            std::cin >> index >> name;
            if (!index.empty() && !name.empty()) {
                students.push_back(Student(index, name));
                saveStudents(students);
            } else {
                std::cout << "Invalid input!\n";
            }
            break;
        }
        case 2:
            std::cout << "Registered students:\n";
            for (const auto& s : students) {
                std::cout << s.indexNumber << " - " << s.name << "\n";
            }
            break;
        case 3: {
            std::string cc, date;
            std::cout << "Enter course code and date (YYYY-MM-DD): ";
            std::cin >> cc >> date;
            sessions.push_back(AttendanceSession(cc, date));
            break;
        }
        case 4: {
            if (students.empty() || sessions.empty()) {
                std::cout << "No students or sessions!\n";
                break;
            }
            std::cout << "Select session:\n";
            for (size_t i = 0; i < sessions.size(); i++) {
                std::cout << i << ". " << sessions[i].courseCode << " - " << sessions[i].date << "\n";
            }
            size_t idx;
            std::cin >> idx;
            if (idx >= sessions.size()) {
                std::cout << "Invalid session!\n";
                break;
            }
            std::cout << "Mark attendance (P/A/L): ";
            char status;
            std::cin >> status;
            if (status != 'P' && status != 'A' && status != 'L') {
                std::cout << "Invalid status!\n";
                break;
            }
            if (!students.empty()) {
                sessions[idx].markAttendance(students[0], status);
                saveSession(sessions[idx]);
            }
            break;
        }
        case 5: {
            if (sessions.empty()) {
                std::cout << "No sessions!\n";
                break;
            }
            std::cout << "Summary:\n";
            for (const auto& session : sessions) {
                int p = 0, a = 0, l = 0;
                for (const auto& att : session.attendance) {
                    if (att.second == 'P') p++;
                    else if (att.second == 'A') a++;
                    else if (att.second == 'L') l++;
                }
                std::cout << session.courseCode << ": P=" << p << ", A=" << a << ", L=" << l << "\n";
            }
            break;
        }
        case 6:
            std::cout << "Exiting...\n";
            break;
        default:
            std::cout << "Invalid choice!\n";
        }
    } while (choice != 6);

    return 0;
}