import json
import os
from statistics import mean
from datetime import datetime

class UniversityGradeSystem:
    def __init__(self):
        self.student_grades = {}
        self.courses = {}
        self.load_data()

    # 数据持久化
    def save_data(self):
        data = {
            'grades': self.student_grades,
            'courses': self.courses,
            'last_updated': datetime.now().isoformat()
        }
        with open('university_data.json', 'w') as f:
            json.dump(data, f, indent=2)

    def load_data(self):
        if os.path.exists('university_data.json'):
            with open('university_data.json') as f:
                data = json.load(f)
                self.student_grades = data.get('grades', {})
                self.courses = data.get('courses', {})
                print(f"上次更新时间: {data.get('last_updated')}")

    # 输入验证装饰器
    def validate_input(func):
        def wrapper(self, *args, **kwargs):
            try:
                return func(self, *args, **kwargs)
            except ValueError as e:
                print(f"输入错误: {e}")
            except Exception as e:
                print(f"发生意外错误: {e}")
            return None
        return wrapper

    @validate_input
    def record_grade(self):
        student_id = input("请输入学号: ").strip()
        if not student_id.isdigit():
            raise ValueError("学号必须为数字")
        
        student_name = input("请输入学生姓名: ").strip()
        if not student_name:
            raise ValueError("姓名不能为空")
        
        course_code = input("请输入课程代码: ").strip().upper()
        if not course_code:
            raise ValueError("课程代码不能为空")
        
        grade = float(input("请输入成绩(0-100): "))
        if not 0 <= grade <= 100:
            raise ValueError("成绩必须在0-100之间")
        
        # 记录课程信息
        if course_code not in self.courses:
            self.courses[course_code] = {
                'name': input(f"请输入{course_code}课程名称: "),
                'credits': int(input("请输入学分: "))
            }
        
        # 记录学生成绩
        if student_id not in self.student_grades:
            self.student_grades[student_id] = {
                'name': student_name,
                'grades': {}
            }
        
        self.student_grades[student_id]['grades'][course_code] = grade
        self.save_data()
        print(f"✅ 成绩记录成功: {student_name}的{course_code}课程成绩为{grade}")

    @validate_input
    def query_grade(self):
        search_type = input("按学号查询(1) 按姓名查询(2): ").strip()
        if search_type == '1':
            student_id = input("请输入学号: ").strip()
            student = self.student_grades.get(student_id)
        elif search_type == '2':
            name = input("请输入姓名: ").strip()
            student = next((s for s in self.student_grades.values() if s['name'] == name), None)
        else:
            raise ValueError("无效的查询方式")
        
        if not student:
            print("⚠️ 未找到该学生记录")
            return
        
        print(f"\n📊 学生成绩单")
        print(f"学号: {student_id if search_type == '1' else 'N/A'}")
        print(f"姓名: {student['name']}")
        print("="*30)
        total_credits = 0
        weighted_sum = 0
        
        for course, grade in student['grades'].items():
            course_info = self.courses.get(course, {'name': course, 'credits': 0})
            print(f"{course} {course_info['name']}: {grade} (学分: {course_info['credits']})")
            total_credits += course_info['credits']
            weighted_sum += grade * course_info['credits']
        
        if total_credits > 0:
            gpa = weighted_sum / total_credits
            print("="*30)
            print(f"GPA: {gpa:.2f} (总学分: {total_credits})")

    def course_statistics(self):
        if not self.courses:
            print("暂无课程数据")
            return
        
        print("\n📈 课程统计分析")
        print("1. 单课程统计")
        print("2. 全部课程统计")
        choice = input("请选择: ").strip()
        
        if choice == '1':
            course_code = input("请输入课程代码: ").strip().upper()
            if course_code not in self.courses:
                print("课程不存在")
                return
            
            grades = [s['grades'][course_code] 
                     for s in self.student_grades.values() 
                     if course_code in s['grades']]
            
            if not grades:
                print("该课程暂无成绩记录")
                return
            
            print(f"\n{course_code} {self.courses[course_code]['name']}统计:")
            print(f"平均分: {mean(grades):.1f}")
            print(f"最高分: {max(grades)}")
            print(f"最低分: {min(grades)}")
            print(f"人数: {len(grades)}")
            
        elif choice == '2':
            print("\n全部课程统计:")
            print("课程代码\t课程名称\t平均分\t最高分\t最低分\t选课人数")
            for code, info in self.courses.items():
                grades = [s['grades'][code] 
                         for s in self.student_grades.values() 
                         if code in s['grades']]
                if grades:
                    print(f"{code}\t{info['name']}\t{mean(grades):.1f}\t"
                          f"{max(grades)}\t{min(grades)}\t{len(grades)}")

    def run(self):
        menu = {
            '1': {'title': '记录成绩', 'func': self.record_grade},
            '2': {'title': '查询成绩', 'func': self.query_grade},
            '3': {'title': '成绩统计', 'func': self.course_statistics},
            '4': {'title': '退出系统', 'func': lambda: (self.save_data(), exit(0))}
        }

        while True:
            print("\n" + "="*40)
            print("🎓 大学生成绩管理系统")
            print("="*40)
            for k, v in menu.items():
                print(f"{k}. {v['title']}")
            print("="*40)

            choice = input("请选择操作: ").strip()
            if choice in menu:
                menu[choice]['func']()
            else:
                print("无效选项，请重新输入！")

if __name__ == "__main__":
    system = UniversityGradeSystem()
    system.run()
