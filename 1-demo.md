Let’s start from the beginning and add the two features to your plugin:

1. **Student Search Box**: A form where you can search by class, section, group, academic year, and roll number.
2. **Displaying Student Profile**: Once a student is found, their details, including image, class, roll number, and parent information, will be displayed.

### 1. **Set Up the Plugin Structure**

First, create a folder called `school-result-manager` inside the `wp-content/plugins/` directory. Inside this folder, create the following files:

* `school-result-manager.php` – Main plugin file
* `includes/` – For backend logic

  * `class-srm-frontend.php` – To handle the student search form and result display
* `templates/` – To store the frontend templates

  * `result-single.php` – Template for displaying the student profile after search
* `assets/` – For images, styles, or JS files (optional for now)

### 2. **Plugin Main File** (`school-result-manager.php`)

Let’s start with the basic setup of the plugin. Here’s how your `school-result-manager.php` file should look:

```php
<?php
/*
Plugin Name: School Result Manager
Plugin URI: http://example.com
Description: Manage school results with GPA and exam info.
Version: 1.0
Author: Your Name
Author URI: http://example.com
License: GPL2
*/

if ( ! defined( 'ABSPATH' ) ) {
    exit; // Exit if accessed directly.
}

// Include required files
require_once plugin_dir_path( __FILE__ ) . 'includes/class-srm-frontend.php';

// Frontend shortcode to display result search form
function srm_result_search_form() {
    ob_start();
    include( plugin_dir_path( __FILE__ ) . 'templates/result-single.php');
    return ob_get_clean();
}
add_shortcode('srm_result_search', 'srm_result_search_form');
```

### 3. **Frontend Logic** (`class-srm-frontend.php`)

Next, we’ll handle the student search logic and displaying the profile once the student is found.

Create `class-srm-frontend.php` inside the `includes/` folder:

```php
<?php

class SRM_Frontend {
    // Function to display student search form
    public static function display_student_search_form() {
        // Check if form is submitted
        if ( isset( $_POST['showStudent'] ) ) {
            $class = sanitize_text_field( $_POST['class'] );
            $roll = sanitize_text_field( $_POST['roll'] );
            // Query the database for student and their profile
            global $wpdb;
            $student = $wpdb->get_row( "SELECT * FROM {$wpdb->prefix}students WHERE class = '$class' AND roll_no = '$roll'" );
            if ( $student ) {
                // Display student profile
                self::display_student_profile( $student );
            } else {
                echo '<p>No student found with this information.</p>';
            }
        } else {
            // Display search form
            ?>
            <form action="" method="POST">
                <div>
                    <label>Class:</label>
                    <select name="class" required>
                        <option value="">Select Class</option>
                        <option value="One">One</option>
                        <option value="Two">Two</option>
                        <!-- Add other classes -->
                    </select>
                </div>
                <div>
                    <label>Roll No:</label>
                    <input type="number" name="roll" required>
                </div>
                <button type="submit" name="showStudent">Search</button>
            </form>
            <?php
        }
    }

    // Function to display student profile after search
    public static function display_student_profile( $student ) {
        ?>
        <div class="student-profile">
            <h2><?php echo $student->name; ?></h2>
            <img src="<?php echo $student->image_url; ?>" alt="Student Image" />
            <table>
                <tr>
                    <th>Class</th>
                    <td><?php echo $student->class; ?></td>
                </tr>
                <tr>
                    <th>Roll No</th>
                    <td><?php echo $student->roll_no; ?></td>
                </tr>
                <tr>
                    <th>Birth Date</th>
                    <td><?php echo $student->birth_date; ?></td>
                </tr>
                <tr>
                    <th>Religion</th>
                    <td><?php echo $student->religion; ?></td>
                </tr>
                <!-- Add more fields as needed -->
            </table>
        </div>
        <?php
    }
}
```

### 4. **Student Data Table** (`students` table)

Before moving forward with the form, we need to ensure that the student data is stored in the WordPress database. You can add a custom database table for students when the plugin is activated.

Let’s create the table when the plugin is activated.

### 5. **Database Setup** (`class-srm-database.php`)

Create a file `class-srm-database.php` inside the `includes/` folder and add the following code to create a table for storing student data.

```php
<?php

class SRM_Database {
    public static function create_student_table() {
        global $wpdb;

        // SQL query to create student data table
        $table_name = $wpdb->prefix . 'students'; // Prefix added for WordPress compatibility
        $sql = "CREATE TABLE $table_name (
            id INT NOT NULL AUTO_INCREMENT,
            name VARCHAR(100) NOT NULL,
            class VARCHAR(10) NOT NULL,
            roll_no INT NOT NULL,
            birth_date DATE,
            religion VARCHAR(50),
            image_url VARCHAR(255),
            PRIMARY KEY(id)
        );";

        require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
        dbDelta( $sql );
    }
}
```

### 6. **Activation Hook** (`class-srm-activator.php`)

Now, let's add the activation hook in the `class-srm-activator.php` to create the `students` table when the plugin is activated.

```php
<?php

class SRM_Activator {
    public static function activate() {
        // Create the student table on activation
        SRM_Database::create_student_table();
    }

    public static function deactivate() {
        // Optional: Drop the table or clean up on deactivation
    }
}
```

### 7. **Displaying the Search Form and Profile**

Now, let’s link everything together in the `school-result-manager.php` file. Add the necessary hooks to display the form and show results.

```php
<?php
/*
Plugin Name: School Result Manager
Plugin URI: http://example.com
Description: Manage school results with GPA and exam info.
Version: 1.0
Author: Your Name
Author URI: http://example.com
License: GPL2
*/

// Include required files
require_once plugin_dir_path( __FILE__ ) . 'includes/class-srm-activator.php';
require_once plugin_dir_path( __FILE__ ) . 'includes/class-srm-database.php';
require_once plugin_dir_path( __FILE__ ) . 'includes/class-srm-frontend.php';

// Activation hook
register_activation_hook( __FILE__, array( 'SRM_Activator', 'activate' ) );

// Frontend shortcode to display result search form
function srm_result_search_form() {
    ob_start();
    SRM_Frontend::display_student_search_form(); // Display the search form
    return ob_get_clean();
}
add_shortcode('srm_result_search', 'srm_result_search_form');
```

### 8. **Test the Plugin**

* **Install the plugin**: Go to the WordPress admin panel, and install the plugin.
* **Activate the plugin**: Upon activation, the table for students will be created in the database.
* **Display the search form**: Use the `[srm_result_search]` shortcode to display the student search form on any page or post.

You will now have a search form where you can search by **class** and **roll number**, and it will show the student profile once found.

---

### Next Steps

Once this is done, we can add additional features like result calculation, displaying semester results, teacher comments, and signatures.

Let me know if you need help with those next steps!
