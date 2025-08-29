==============================================================================================
T
```
  
  _              _        ___               
 | |   ___  __ _(_)_ _   | _ \__ _ __ _ ___ 
 | |__/ _ \/ _` | | ' \  |  _/ _` / _` / -_)
 |____\___/\__, |_|_||_| |_| \__,_\__, \___|
           |___/                  |___/
```
---
```cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Data.SqlClient;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using static FinalTermProject.Home;
using System.Data.SqlClient;

namespace FinalTermProject
{
    public partial class LoginForm : Form
    {

        int xLast, yLast;
        bool mouseDown;

        public LoginForm()
        {
            InitializeComponent();
        }

        private void buttonClose(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void MinimiseButton(object sender, EventArgs e)
        {
            this.WindowState = FormWindowState.Minimized;
        }

        private void loginButton_Click(object sender, EventArgs e)
        {
            //@ or //(double slash)
            string username = textBox_username.Text.Trim();
            string password = textBox_pass.Text.Trim();

            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
            {
                MessageBox.Show("Please enter both username and password.", "Missing Data", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            using (SqlConnection conn = new SqlConnection(DB.connectionString))
            {
                string query = "SELECT COUNT(*) FROM Admins WHERE Username = @Username AND Password = @Password";
                SqlCommand cmd = new SqlCommand(query, conn);
                cmd.Parameters.AddWithValue("@Username", username);
                cmd.Parameters.AddWithValue("@Password", password); 

                conn.Open();
                int count = (int)cmd.ExecuteScalar();
                conn.Close();

                if (count > 0)
                {
                    Home home = new Home();
                    this.Hide();
                    home.Show();
                }
                else
                {
                    MessageBox.Show("Invalid username or password.", "Login Failed", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }

        }

        private void LoginForm_MouseMove(object sender, MouseEventArgs e)
        {
            if (mouseDown)
            {
                int newX = this.Left + (e.X - xLast);
                int newY = this.Top + (e.Y - yLast);
                this.Location = new Point(newX, newY);
            }
        }

        private void LoginForm_MouseUp(object sender, MouseEventArgs e)
        {
            mouseDown = false;
        }

        private void LoginForm_MouseDown(object sender, MouseEventArgs e)
        {
            mouseDown = true;

            xLast = e.X;
            yLast = e.Y;
        }

        private void bunifuFlatButton1_Click(object sender, EventArgs e)
        {
            MessageBox.Show("This section is not currently available!");
        }

        private void label2_Click(object sender, EventArgs e)
        {
             
        }

        private void textBox_username_TextChanged(object sender, EventArgs e)
        {
            
            if (string.IsNullOrEmpty(textBox_username.Text))
            {
                
                MessageBox.Show("You must enter a username!");
            }

        }

        private void label3_Click(object sender, EventArgs e)
        {

        }

        private void label1_Click(object sender, EventArgs e)
        {

        }

        private void LoginForm_Load(object sender, EventArgs e)
        {

        }

        private void textBox_pass_TextChanged(object sender, EventArgs e)
        {
            
            if (string.IsNullOrEmpty(textBox_username.Text))
            {
                
                MessageBox.Show("You Must Enter a Username!");
            }
        }

        private void panel1_Paint(object sender, PaintEventArgs e)
        {

        }

        private void label4_Click(object sender, EventArgs e)
        {

        }

        private void panel3_Paint(object sender, PaintEventArgs e)
        {

        }

        private void label5_Click(object sender, EventArgs e)
        {

        }

        private void label6_Click(object sender, EventArgs e)
        {

        }

        private void pictureBox3_Click(object sender, EventArgs e)
        {

        }

        private void pictureBox5_Click(object sender, EventArgs e)
        {

        }

        private void textBox_pass_KeyPress(object sender, KeyPressEventArgs e)
        {
            if (e.KeyChar == Convert.ToChar(Keys.Enter))
                loginButton_Click(sender, e);
        }

    }
}

```
