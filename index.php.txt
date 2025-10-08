<?php
// database configuration
$host = "localhost";      
$user = "root";           
$pass = "";               
$db   = "studenthub";     

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Database Connection failed: " . htmlspecialchars($conn->connect_error));
}

$msg = "";
$editStudent = null; // for storing data while editing

// insert
if (isset($_POST['add_student'])) {
    $stmt = $conn->prepare("INSERT INTO students (student_id, name, email, course, year) VALUES (?, ?, ?, ?, ?)");
    $stmt->bind_param("isssi", $_POST['student_id'], $_POST['name'], $_POST['email'], $_POST['course'], $_POST['year']);
    if ($stmt->execute()) $msg = "Student added successfully!";
    else $msg = "Error: " . htmlspecialchars($stmt->error);
    $stmt->close();
}

// update
if (isset($_POST['update_student'])) {
    $stmt = $conn->prepare("UPDATE students SET name=?, email=?, course=?, year=? WHERE student_id=?");
    $stmt->bind_param("sssii", $_POST['name'], $_POST['email'], $_POST['course'], $_POST['year'], $_POST['student_id']);
    if ($stmt->execute()) $msg = "Student updated successfully!";
    else $msg = "Error: " . htmlspecialchars($stmt->error);
    $stmt->close();
}

// delete
if (isset($_POST['delete_student'])) {
    $stmt = $conn->prepare("DELETE FROM students WHERE student_id=?");
    $stmt->bind_param("i", $_POST['student_id']);
    if ($stmt->execute()) $msg = "Student deleted successfully!";
    else $msg = "Error: " . htmlspecialchars($stmt->error);
    $stmt->close();
}

// load student for edit 

if (isset($_GET['edit'])) {
    $id = intval($_GET['edit']);
    $res = $conn->prepare("SELECT * FROM students WHERE student_id=?");
    $res->bind_param("i", $id);
    $res->execute();
    $editStudent = $res->get_result()->fetch_assoc();
    $res->close();
}

// fetch student for search and update
$students = $conn->query("SELECT * FROM students ORDER BY student_id ASC");
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>StudentHub Portal</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');
        
        * { margin: 0; padding: 0; box-sizing: border-box; }
        
        body { 
            font-family: 'Poppins', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
            min-height: 100vh;
            padding: 20px;
            position: relative;
            overflow-x: hidden;
        }
        
        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><defs><pattern id="grain" width="100" height="100" patternUnits="userSpaceOnUse"><circle cx="50" cy="50" r="1" fill="%23ffffff" opacity="0.1"/></pattern></defs><rect width="100" height="100" fill="url(%23grain)"/></svg>');
            pointer-events: none;
            z-index: -1;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 25px;
            box-shadow: 0 25px 50px rgba(0,0,0,0.15), 0 0 0 1px rgba(255,255,255,0.1);
            backdrop-filter: blur(10px);
            overflow: hidden;
            position: relative;
        }
        
        .container::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 4px;
            background: linear-gradient(90deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4, #feca57);
            animation: shimmer 3s ease-in-out infinite;
        }
        
        @keyframes shimmer {
            0%, 100% { transform: translateX(-100%); }
            50% { transform: translateX(100%); }
        }
        
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
            color: white;
            padding: 40px;
            text-align: center;
            position: relative;
            overflow: hidden;
        }
        
        .header::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%);
            animation: rotate 20s linear infinite;
        }
        
        @keyframes rotate {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }
        
        .header h1 {
            font-size: 3.2em;
            font-weight: 700;
            margin-bottom: 15px;
            text-shadow: 2px 2px 20px rgba(0,0,0,0.3);
            position: relative;
            z-index: 2;
            letter-spacing: -1px;
        }
        
        .header p {
            font-size: 1.1em;
            font-weight: 300;
            opacity: 0.9;
            position: relative;
            z-index: 2;
        }
        
        .content {
            padding: 40px;
            background: linear-gradient(145deg, #f8f9fa 0%, #e9ecef 100%);
        }
        
        .form-section {
            background: linear-gradient(145deg, #ffffff 0%, #f8f9fa 100%);
            padding: 35px;
            border-radius: 20px;
            margin-bottom: 35px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.1), inset 0 1px 0 rgba(255,255,255,0.6);
            border: 1px solid rgba(255,255,255,0.2);
            position: relative;
            overflow: hidden;
        }
        
        .form-section::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 3px;
            background: linear-gradient(90deg, #667eea, #764ba2, #f093fb);
        }
        
        .form-section h2 {
            font-size: 1.8em;
            font-weight: 600;
            color: #2c3e50;
            margin-bottom: 25px;
            position: relative;
        }
        
        .form-row {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }
        
        .input-group {
            position: relative;
        }
        
        input[type="text"], input[type="email"], input[type="number"] {
            width: 100%;
            padding: 15px 20px;
            border: 2px solid #e1e8ed;
            border-radius: 15px;
            font-size: 15px;
            font-weight: 400;
            background: rgba(255,255,255,0.8);
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            backdrop-filter: blur(5px);
        }
        
        input:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1), 0 8px 25px rgba(102, 126, 234, 0.15);
            transform: translateY(-2px);
            background: rgba(255,255,255,0.95);
        }
        
        .btn {
            padding: 15px 30px;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            font-size: 15px;
            font-weight: 600;
            margin: 8px;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            position: relative;
            overflow: hidden;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        
        .btn::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
            transition: left 0.6s;
        }
        
        .btn:hover::before {
            left: 100%;
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            box-shadow: 0 8px 25px rgba(102, 126, 234, 0.3);
        }
        
        .btn-success {
            background: linear-gradient(135deg, #4ecdc4 0%, #44a08d 100%);
            color: white;
            box-shadow: 0 8px 25px rgba(78, 205, 196, 0.3);
        }
        
        .btn-danger {
            background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
            color: white;
            box-shadow: 0 8px 25px rgba(255, 107, 107, 0.3);
        }
        
        .btn-secondary {
            background: linear-gradient(135deg, #95a5a6 0%, #7f8c8d 100%);
            color: white;
            box-shadow: 0 8px 25px rgba(149, 165, 166, 0.3);
        }
        
        .btn:hover {
            transform: translateY(-3px) scale(1.02);
            box-shadow: 0 15px 35px rgba(0,0,0,0.2);
        }
        
        .btn:active {
            transform: translateY(-1px) scale(0.98);
        }
        
        table {
            width: 100%;
            border-collapse: separate;
            border-spacing: 0;
            border-radius: 20px;
            overflow: hidden;
            box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            background: white;
        }
        
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px 15px;
            font-weight: 600;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            font-size: 14px;
        }
        
        td {
            padding: 18px 15px;
            border-bottom: 1px solid #f1f3f4;
            font-weight: 400;
            transition: all 0.3s ease;
        }
        
        tr:hover {
            background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
            transform: scale(1.01);
        }
        
        tr:last-child td {
            border-bottom: none;
        }
        
        .msg {
            padding: 20px 25px;
            margin: 25px 0;
            border-radius: 15px;
            background: linear-gradient(135deg, #4ecdc4 0%, #44a08d 100%);
            color: white;
            font-weight: 500;
            box-shadow: 0 10px 30px rgba(78, 205, 196, 0.3);
            border-left: 5px solid rgba(255,255,255,0.3);
            animation: slideIn 0.5s ease-out;
        }
        
        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(-20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        .action-buttons {
            display: flex;
            gap: 8px;
            justify-content: center;
        }
        
        @media (max-width: 768px) {
            .container { margin: 10px; border-radius: 15px; }
            .header { padding: 25px; }
            .header h1 { font-size: 2.2em; }
            .content { padding: 25px; }
            .form-section { padding: 25px; }
            .form-row { grid-template-columns: 1fr; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🎓 Student Management System</h1>
            <p><strong>Created by JASH JOSHI-24CS030</strong></p>
        </div>
        
        <div class="content">
            <?php if (!empty($msg)) echo "<div class='msg'>$msg</div>"; ?>
            
            <div class="form-section">
                <h2><?= $editStudent ? "📝 Edit Student (ID: {$editStudent['student_id']})" : "➕ Add New Student" ?></h2>
                <form method="POST">
                    <div class="form-row">
                        <div class="input-group">
                            <input type="number" name="student_id" placeholder="🆔 Student ID" value="<?= $editStudent['student_id'] ?? '' ?>" <?= $editStudent ? 'readonly' : 'required' ?>>
                        </div>
                        <div class="input-group">
                            <input type="text" name="name" placeholder="👤 Full Name" value="<?= $editStudent['name'] ?? '' ?>" required>
                        </div>
                    </div>
                    <div class="form-row">
                        <div class="input-group">
                            <input type="email" name="email" placeholder="📧 Email Address" value="<?= $editStudent['email'] ?? '' ?>" required>
                        </div>
                        <div class="input-group">
                            <input type="text" name="course" placeholder="📚 Course" value="<?= $editStudent['course'] ?? '' ?>" required>
                        </div>
                        <div class="input-group">
                            <input type="number" name="year" placeholder="📅 Academic Year" value="<?= $editStudent['year'] ?? '' ?>" required min="1" max="4">
                        </div>
                    </div>
                    
                    <?php if ($editStudent): ?>
                        <button type="submit" name="update_student" class="btn btn-success">Update Student</button>
                        <a href="studenthub.php" class="btn btn-secondary">Cancel</a>
                    <?php else: ?>
                        <button type="submit" name="add_student" class="btn btn-primary">Add Student</button>
                    <?php endif; ?>
                </form>
            </div>

            <div class="form-section">
                <h2>📄 All Students</h2>
                <table>
                    <thead>
                        <tr>
                            <th>🆔 ID</th>
                            <th>👤 Name</th>
                            <th>📧 Email</th>
                            <th>📚 Course</th>
                            <th>📅 Year</th>
                            <th>⚙ Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php while ($row = $students->fetch_assoc()): ?>
                        <tr>
                            <td><?= htmlspecialchars($row['student_id']) ?></td>
                            <td><?= htmlspecialchars($row['name']) ?></td>
                            <td><?= htmlspecialchars($row['email']) ?></td>
                            <td><?= htmlspecialchars($row['course']) ?></td>
                            <td><?= htmlspecialchars($row['year']) ?></td>
                            <td>
                                <div class="action-buttons">
                                    <form method="GET" style="display:inline;">
                                        <input type="hidden" name="edit" value="<?= $row['student_id'] ?>">
                                        <button type="submit" class="btn btn-primary">✏ Edit</button>
                                    </form>
                                    <form method="POST" style="display:inline;" onsubmit="return confirm('⚠ Are you sure you want to delete this student?')">
                                        <input type="hidden" name="student_id" value="<?= $row['student_id'] ?>">
                                        <button type="submit" name="delete_student" class="btn btn-danger">🗑 Remove</button>
                                    </form>
                                </div>
                            </td>
                        </tr>
                        <?php endwhile; ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</body>
</html>
<?php $conn->close(); ?>