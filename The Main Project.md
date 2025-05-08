__________________________________________________________________________________________________________


```
  
 _____ _           ___  ___      _        ______          _           _   
|_   _| |          |  \/  |     (_)       | ___ \        (_)         | |  
  | | | |__   ___  | .  . | __ _ _ _ __   | |_/ / __ ___  _  ___  ___| |_ 
  | | | '_ \ / _ \ | |\/| |/ _` | | '_ \  |  __/ '__/ _ \| |/ _ \/ __| __|
  | | | | | |  __/ | |  | | (_| | | | | | | |  | | | (_) | |  __/ (__| |_ 
  \_/ |_| |_|\___| \_|  |_/\__,_|_|_| |_| \_|  |_|  \___/| |\___|\___|\__|
                                                        
```

```cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Data.SqlClient;




namespace FinalTermProject
{
    public partial class Home : Form
    {

        int xLast, yLast, quantity;
        bool mouseDown;
        double total;


        //----------------------------------------HOME-----------------------------------------

        public Home()
        {
            InitializeComponent();
            this.Load += new System.EventHandler(this.AddRemoveAdmin_Load);
            this.Load += new EventHandler(YourForm_Load);
            this.Load += new EventHandler(UpdateMembers_Load);
            this.Load += new EventHandler(ProductPurchaseForm_Load);
            this.Load += new System.EventHandler(this.ProductEditForm_Load);



            this.textBox1.TextChanged += new System.EventHandler(this.textBox1_TextChanged);
            this.textBox5.TextChanged += new System.EventHandler(this.textBox5_TextChanged);

            
            panel_Home.Visible = true;
            panel_AddMem.Visible = true;
            panel_Mem_Info.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_Admin.Visible = false;
            button_New_Members.Show();

        }

        private void label1_Click(object sender, EventArgs e)
        {
            panel_Home.Visible = true;
            panel_AddMem.Visible = false;
            panel_Mem_Info.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_Admin.Visible = false;
        }


        public static class DB
        {
            //Data Source=DESKTOP-8LAI86N;Initial Catalog=GymDB;Integrated Security=True;Trust Server Certificate=True
            public static string connectionString = "Data Source=DESKTOP-8LAI86N;Initial Catalog=GymDB;Integrated Security=True";
        }



        private void buttonClose(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void minimiseTab(object sender, EventArgs e)
        {
            this.WindowState = FormWindowState.Minimized;
        }



        private void headerPanel_MouseDown(object sender, MouseEventArgs e)
        {
            mouseDown = true;

            xLast = e.X;
            yLast = e.Y;
        }

        private void headerPanel_MouseMove(object sender, MouseEventArgs e)
        {
            if (mouseDown)
            {
                int newX = this.Left + (e.X - xLast);
                int newY = this.Top + (e.Y - yLast);
                this.Location = new Point(newX, newY);
            }
        }

        private void headerPanel_MouseUp(object sender, MouseEventArgs e)
        {
            mouseDown = false;
        }


        private void button14_Click(object sender, EventArgs e)//image gallery
        {

            //panel_Home.Visible = false;
            //panel_AddMem.Visible = false;
            //panel_Purchase_Product.Visible = false;
            //panel_Admin.Visible = true;
            //panel_Admin.BringToFront();  


            try
            {
                System.Diagnostics.Process.Start("explorer.exe", @"C:\Pictures");
            }
            catch (Win32Exception win32Exception)
            {
                Console.WriteLine(win32Exception.Message);
            
            }

        }
     


        //-----------------------------------END OF HOME------------------------------------------



        //----------------------------------ADD MEMBER--------------------------------------------


        private void ad_mem_click(object sender, EventArgs e)
        {
            panel_AddMem.Visible = true;
            panel_Home.Visible = false;
            panel_Mem_Info.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_Admin.Visible = false;

        }

        private void buttonSubmit_Click(object sender, EventArgs e)
        {
            using (SqlConnection conn = new SqlConnection(DB.connectionString))
            {
                try
                {
                    conn.Open();

                    string getMaxIdQuery = "SELECT ISNULL(MAX(CAST(Id AS INT)), 0) FROM Members";
                    SqlCommand getMaxIdCmd = new SqlCommand(getMaxIdQuery, conn);
                    int newId = (int)getMaxIdCmd.ExecuteScalar() + 1;

                    string formattedId = newId.ToString("D3");

                    textBox_Id.Text = formattedId;


                    // 3. تنفيذ عملية الإدخال
                    string query = "INSERT INTO Members (Id, Name, Gender, Age, Height, Weight, Contact, Batch, MemberType, SubscriptionType, Price, StartDate, EndDate) " +
                                   "VALUES (@Id, @Name, @Gender, @Age, @Height, @Weight, @Contact, @Batch, @MemberType, @SubscriptionType, @Price, @StartDate, @EndDate)";

                    SqlCommand cmd = new SqlCommand(query, conn);

                    cmd.Parameters.AddWithValue("@Id", formattedId);
                    cmd.Parameters.AddWithValue("@Name", textName.Text);
                    cmd.Parameters.AddWithValue("@Gender", comboBox_Gender.SelectedItem.ToString());
                    cmd.Parameters.AddWithValue("@Age", int.Parse(textBox_Age.Text));
                    cmd.Parameters.AddWithValue("@Height", float.Parse(textHeight.Text));
                    cmd.Parameters.AddWithValue("@Weight", float.Parse(textWeight.Text));
                    cmd.Parameters.AddWithValue("@Contact", textContact.Text);
                    cmd.Parameters.AddWithValue("@Batch", comboBatch.SelectedItem.ToString());
                    cmd.Parameters.AddWithValue("@MemberType", comboMember.SelectedItem.ToString());
                    cmd.Parameters.AddWithValue("@SubscriptionType", comboSubscription_Type.SelectedItem.ToString());
                    cmd.Parameters.AddWithValue("@Price", float.Parse(Price_Lable.Text));

                    // ✅ حساب تاريخ الانتهاء تلقائيًا بعد 30 يوم من تاريخ البدء
                    DateTime startDate = dateTimePickerStart.Value.Date;
                    DateTime endDate = startDate.AddDays(30);
                    dateTimePickerEnd.Value = endDate;
                    cmd.Parameters.AddWithValue("@StartDate", startDate);
                    cmd.Parameters.AddWithValue("@EndDate", endDate);

                    // عرض التاريخ في الحقل
                    

                    cmd.ExecuteNonQuery();
                    MessageBox.Show("Member Added Successfully");

                    conn.Close();

                    LoadMembers(); // تحديث الجدول

                    textBox_Id.Text = formattedId; // يظهر الـ ID الجديد في واجهة المستخدم
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Error: " + ex.Message);
                    conn.Close();
                }

            }
        }
        private void LoadMembers()
        {
            using (SqlConnection conn = new SqlConnection(DB.connectionString))
            {
                try
                {
                    SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Members", conn);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    dataGridView1.DataSource = dt;
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Error loading members: " + ex.Message);
                }
            }
        }
        private void YourForm_Load(object sender, EventArgs e)
        {
            LoadMembers();
        }





        private void button_Clear_2_Click(object sender, EventArgs e)
        {
            DialogResult result = MessageBox.Show("هل أنت متأكد من حذف جميع البيانات؟", "تأكيد المسح", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                // تفريغ TextBoxes
                textBox_Id.Clear();
                textName.Clear();
                textBox_Age.Clear();
                textHeight.Clear();
                textWeight.Clear();
                textContact.Clear();

               
                comboBox_Gender.SelectedIndex = -1;
                comboBatch.SelectedIndex = -1;
                comboMember.SelectedIndex = -1;
                comboSubscription_Type.SelectedIndex = -1;

              
                Price_Lable.Text = "";

                // إعادة تعيين التاريخ
                dateTimePickerStart.Value = DateTime.Now;
                dateTimePickerEnd.Value = DateTime.Now;
            }
            
        }

        //-----------------------------------END OF ADD MEMBER-----------------------------------------




        //--------------------------------------MEMBER INFO---------------------------------------------

        private void b1_Click(object sender, EventArgs e)
        {
            panel_Mem_Info.Visible = true;
            panel_Home.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_AddMem.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Admin.Visible = false;

            LoadTotalMembers(); // عرض عدد كل الأعضاء في الجيم
        }

        private void LoadTotalMembers()
        {
            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();
                    string query = "SELECT COUNT(Id) FROM Members"; // عدّ الـ IDs
                    SqlCommand cmd = new SqlCommand(query, con);
                    int total = (int)cmd.ExecuteScalar();

                    lblTotalMembers.Text = "All Members : " + total.ToString();
                }
                catch (Exception ex)
                {
                    MessageBox.Show("حصل خطأ أثناء تحميل عدد الأعضاء: " + ex.Message);
                }
            }
        }

        private void SearchMember()
        {
            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();

                    string query = "SELECT * FROM Members WHERE 1=1";
                    List<SqlParameter> parameters = new List<SqlParameter>();

                    if (!string.IsNullOrWhiteSpace(textBox1.Text))
                    {
                        query += " AND Name LIKE @name";
                        parameters.Add(new SqlParameter("@name", "%" + textBox1.Text + "%"));
                    }

                    if (!string.IsNullOrWhiteSpace(textBox5.Text))
                    {
                        query += " AND Id LIKE @id";
                        parameters.Add(new SqlParameter("@id", "%" + textBox5.Text + "%"));
                    }

                    using (SqlCommand cmd = new SqlCommand(query, con))
                    {
                        cmd.Parameters.AddRange(parameters.ToArray());

                        SqlDataAdapter da = new SqlDataAdapter(cmd);
                        DataTable dt = new DataTable();
                        da.Fill(dt);
                        dataGridView1.DataSource = dt;

                        // مش هنعدل الـ Label هنا، عشان ده بحث مش عدد كل الأعضاء
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Error: " + ex.Message);
                }
            }
        }

        private void textBox1_TextChanged(object sender, EventArgs e)
        {
            SearchMember();
        }

        private void textBox5_TextChanged(object sender, EventArgs e)
        {
            SearchMember();
        }

        private void buttonRefresh_Click(object sender, EventArgs e)
        {
            LoadMembers(); // دي مفروض بتعمل إعادة تحميل لكل الأعضاء
            LoadTotalMembers(); // وهنا بنجيب العدد الكلي بعد الريفريش
        }



        //-----------------------------------END OF MEMBER INFO-----------------------------------------





        //--------------------------------------UPDATE MEMBER-------------------------------------------

        private void b3_Click(object sender, EventArgs e) //Update Frame button
        {
            panel_Mem_Info.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_Home.Visible = false;
            panel_AddMem.Visible = false;
            panel_Update_Member.Visible = true;
            panel_Admin.Visible = false;
        }

        private void buttonMemSearch_Click(object sender, EventArgs e) //SEARCH button in Update info Panel
        {
            if (string.IsNullOrWhiteSpace(textBox_Up_Id.Text))
            {
                MessageBox.Show("من فضلك أدخل رقم ID للبحث.");
                return;
            }

            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();
                    string query = "SELECT * FROM Members WHERE Id = @Id";
                    SqlCommand cmd = new SqlCommand(query, con);
                    cmd.Parameters.AddWithValue("@Id", int.Parse(textBox_Up_Id.Text));

                    SqlDataAdapter da = new SqlDataAdapter(cmd);
                    DataTable dt = new DataTable();
                    da.Fill(dt);

                    if (dt.Rows.Count > 0)
                    {
                        DataRow row = dt.Rows[0];
                        textBox_Up_name.Text = row["Name"].ToString();
                        comboBox_Up_Gen.Text = row["Gender"].ToString();
                        textBox_Up_Age.Text = row["Age"].ToString();
                        textBox_Up_Height.Text = row["Height"].ToString();
                        textBox_Up_Weight.Text = row["Weight"].ToString();
                        textBox_Up_Contact.Text = row["Contact"].ToString();
                        comboBox_Up_batch.Text = row["Batch"].ToString();
                        comboBox_Up_Mem.Text = row["MemberType"].ToString();
                        price_up.Text = row["Price"].ToString();

                        // تحميل التاريخ
                        dateTimePicker1.Value = Convert.ToDateTime(row["StartDate"]);

                        // عرض نتيجة البحث فقط في الجدول
                        dataGrid_Update_List.DataSource = dt;
                    }
                    else
                    {
                        MessageBox.Show("لا يوجد عضو بهذا الرقم.");
                        dataGrid_Update_List.DataSource = null;
                        ClearUpdateFields();
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show("حدث خطأ أثناء البحث: " + ex.Message);
                }
            }
        }

        private void buttonUpdate_Click(object sender, EventArgs e) //Update Button in Update Info Panel
        {
            if (string.IsNullOrWhiteSpace(textBox_Up_Id.Text))
            {
                MessageBox.Show("من فضلك أدخل ID العضو أولاً.");
                return;
            }

            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();
                    string query = @"UPDATE Members SET 
                            Name = @Name,
                            Gender = @Gender,
                            Age = @Age,
                            Height = @Height,
                            Weight = @Weight,
                            Contact = @Contact,
                            Batch = @Batch,
                            MemberType = @MemberType,
                            Price = @Price,
                            StartDate = @StartDate
                        WHERE Id = @Id";

                    SqlCommand cmd = new SqlCommand(query, con);
                    cmd.Parameters.AddWithValue("@Id", int.Parse(textBox_Up_Id.Text));
                    cmd.Parameters.AddWithValue("@Name", textBox_Up_name.Text);
                    cmd.Parameters.AddWithValue("@Gender", comboBox_Up_Gen.Text);
                    cmd.Parameters.AddWithValue("@Age", int.Parse(textBox_Up_Age.Text));
                    cmd.Parameters.AddWithValue("@Height", float.Parse(textBox_Up_Height.Text));
                    cmd.Parameters.AddWithValue("@Weight", float.Parse(textBox_Up_Weight.Text));
                    cmd.Parameters.AddWithValue("@Contact", textBox_Up_Contact.Text);
                    cmd.Parameters.AddWithValue("@Batch", comboBox_Up_batch.Text);
                    cmd.Parameters.AddWithValue("@MemberType", comboBox_Up_Mem.Text);
                    cmd.Parameters.AddWithValue("@Price", float.Parse(price_up.Text));
                    cmd.Parameters.AddWithValue("@StartDate", dateTimePicker1.Value);

                    cmd.ExecuteNonQuery();
                    MessageBox.Show("تم تعديل بيانات العضو بنجاح.");

                    LoadMembersToGrid();   // تحديث الجدول
                    ClearUpdateFields();   // تفريغ الحقول
                }
                catch (Exception ex)
                {
                    MessageBox.Show("خطأ أثناء التعديل: " + ex.Message);
                }
            }
        }

        private void buttonDelete_Click(object sender, EventArgs e) //DELETE Button in Update Info Panel 
        {
            if (string.IsNullOrWhiteSpace(textBox_Up_Id.Text))
            {
                MessageBox.Show("أدخل ID العضو المراد حذفه.");
                return;
            }

            DialogResult result = MessageBox.Show("هل أنت متأكد من حذف هذا العضو؟", "تأكيد الحذف", MessageBoxButtons.YesNo, MessageBoxIcon.Warning);
            if (result == DialogResult.No)
                return;

            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();
                    string query = "DELETE FROM Members WHERE Id = @Id";
                    SqlCommand cmd = new SqlCommand(query, con);
                    cmd.Parameters.AddWithValue("@Id", int.Parse(textBox_Up_Id.Text));
                    cmd.ExecuteNonQuery();

                    MessageBox.Show("تم حذف العضو بنجاح.");
                    ClearUpdateFields(); 
                    LoadMembersToGrid(); 
                }
                catch (Exception ex)
                {
                    MessageBox.Show("خطأ أثناء الحذف: " + ex.Message);
                }
            }
        }

        private void ClearUpdateFields()
        {
            textBox_Up_Id.Clear();
            textBox_Up_name.Clear();
            comboBox_Up_Gen.SelectedIndex = -1;
            textBox_Up_Age.Clear();
            textBox_Up_Height.Clear();
            textBox_Up_Weight.Clear();
            textBox_Up_Contact.Clear();
            comboBox_Up_batch.SelectedIndex = -1;
            comboBox_Up_Mem.SelectedIndex = -1;
            price_up.Clear();
            dateTimePicker1.Value = DateTime.Today; 
        }

        private void LoadMembersToGrid()
        {
            using (SqlConnection con = new SqlConnection(DB.connectionString))
            {
                try
                {
                    con.Open();
                    string query = "SELECT * FROM Members";
                    SqlDataAdapter da = new SqlDataAdapter(query, con);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    dataGrid_Update_List.DataSource = dt;
                }
                catch (Exception ex)
                {
                    MessageBox.Show("خطأ في تحميل البيانات: " + ex.Message);
                }
            }
        }

        private void UpdateMembers_Load(object sender, EventArgs e)
        {
            LoadMembersToGrid();
        }

        private void buttonLoad_Click(object sender, EventArgs e) //REFRESH Button in Update info Panel
        {
            LoadMembersToGrid();
            ClearUpdateFields();
        }

        private void dataGrid_Update_List_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
       
        }


        //-----------------------------------END OF UPDATE INFO-----------------------------------------





        //-----------------------------------PURCHASE PRODUCT-----------------------------------------

        private void b4_Click(object sender, EventArgs e)//PRODUCT PURCHASE Button
        {
           

            panel_Mem_Info.Visible = false;
            panel_Purchase_Product.Visible = true;
            panel_Home.Visible = false;
            panel_AddMem.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Admin.Visible = false;
        }

        //TIMER
        private void timer1_Tick(object sender, EventArgs e)
        {
            if (label_Notice.Left < -250)
            {
                label_Notice.Left = 300;
            }
            else
            {
                label_Notice.Left -= 5;
            }

        }

        private void button_Prod_Search_Click(object sender, EventArgs e)//Product Search
        {
        }

        private void LoadProductsToGrid()
        {
            
        }
       

        private void button_Purchase_Click(object sender, EventArgs e) //back 
        {
            groupBox_Prod.Visible = false;
            groupBox_Bill.Visible = true; 

        }



        private void buttonRefresh_P_Click(object sender, EventArgs e) //REFRESH PRODUCT Button
        {
           

        }



        private void button8_Click(object sender, EventArgs e)//UPDATE PRODUCT Button
        {
           
        }



        private void button6_Click(object sender, EventArgs e)//ADD PRODUCT Button
        {
           
        }

        private void button_Edit_Prod_Click(object sender, EventArgs e) //ADD/EDIT PRODUCT Button
        {
            groupBox_Prod.Visible = true;
        }

        private void bunifuFlatButton3_Click(object sender, EventArgs e)//CLOSE BUTTON
        {
            groupBox_Prod.Visible = false;
        } 
        private void ReloadPurchaseData()
        {
            try
            {
                using (SqlConnection conn = new SqlConnection(DB.connectionString))
                {
                    conn.Open();

                    string query = "SELECT * FROM Products";
                    SqlDataAdapter da = new SqlDataAdapter(query, conn);
                    DataTable dt = new DataTable();
                    da.Fill(dt);

                    dataGrid_Purchase.DataSource = dt;
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Error loading data: " + ex.Message);
            }
        }
        private void ProductEditForm_Load(object sender, EventArgs e)
        {
            ReloadPurchaseData(); 
        }


        //-----------------------------------END OF PURCHASE PRODUCT-----------------------------------------




        //----------------------------------------BILL SECTION-----------------------------------------------

        private void text_pID_TextChanged(object sender, EventArgs e)//PRODUCT ID (BILL SECTION)
        {

           
        }


        private void text_Quantity_TextChanged(object sender, EventArgs e)//QUANTITY Textbox (BILL SECTION)
        {

           

        }


        private void button2_Click_1(object sender, EventArgs e) //ADD Items (BILL SECTIOIN)
        {
            try
            {
                using (SqlConnection conn = new SqlConnection(DB.connectionString))
                {
                    conn.Open();
                    decimal rate = decimal.Parse(textBox_Rate.Text);
                    int quantity = int.Parse(text_Quantity.Text);
                    decimal cost = rate * quantity;

                    string query = "INSERT INTO Products (ProductID, ProductName, Rate, Quantity, Cost) VALUES (@ID, @Name, @Rate, @Quantity, @Cost)";
                    SqlCommand cmd = new SqlCommand(query, conn);
                    cmd.Parameters.AddWithValue("@ID", int.Parse(text_ID.Text));
                    cmd.Parameters.AddWithValue("@Name", box_name.Text);
                    cmd.Parameters.AddWithValue("@Rate", rate);
                    cmd.Parameters.AddWithValue("@Quantity", quantity);
                    cmd.Parameters.AddWithValue("@Cost", cost);

                    cmd.ExecuteNonQuery();
                    MessageBox.Show("Item added successfully!");
                    LoadProductData(); 
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Error: " + ex.Message);
            }
        }

        private void button9_Click(object sender, EventArgs e)//BILL Button (BILL SECTION)
        {
           

        }

        private void button10_Click(object sender, EventArgs e)//REMOVE Item (BILL SECTION)
        {
            if (dataGridView_Bill.SelectedRows.Count > 0)
            {
                int productId = Convert.ToInt32(dataGridView_Bill.SelectedRows[0].Cells["ProductID"].Value);
                try
                {
                    using (SqlConnection conn = new SqlConnection(DB.connectionString))
                    {
                        conn.Open();
                        string query = "DELETE FROM Products WHERE ProductID = @ID";
                        SqlCommand cmd = new SqlCommand(query, conn);
                        cmd.Parameters.AddWithValue("@ID", productId);
                        cmd.ExecuteNonQuery();
                        MessageBox.Show("Item removed successfully!");
                        LoadProductData(); // تحديث الداتا جريد
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Error: " + ex.Message);
                }
            }
            else
            {
                MessageBox.Show("Please select a row to remove.");
            }
        }
        


        private void button16_Click(object sender, EventArgs e) //REFRESH Button (BILL SECTION)
        {
            LoadProductData();
        }
 

        private void LoadProductData()
        {
            try
            {
                using (SqlConnection conn = new SqlConnection(DB.connectionString))
                {
                    conn.Open();
                    string query = "SELECT * FROM Products";
                    SqlDataAdapter da = new SqlDataAdapter(query, conn);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    dataGridView_Bill.DataSource = dt;
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Error loading data: " + ex.Message);
            }
        }
      



        private void ProductPurchaseForm_Load(object sender, EventArgs e)
        {
            LoadProductData(); // دي الدالة اللي بتعرض البيانات في الجريد
        }


        //-----------------------------------END OF BILL SECTION-----------------------------------------



        //-------------------------------------ADD/REMOVE ADMIN------------------------------------------

        private void b5_Click(object sender, EventArgs e)//ADD/REMOVE Admin Button
        {


            panel_Mem_Info.Visible = false;
            panel_Purchase_Product.Visible = false;
            panel_Home.Visible = false;
            panel_AddMem.Visible = false;
            panel_Update_Member.Visible = false;
            panel_Admin.Visible = true;


        }

        private void button4_Click(object sender, EventArgs e)//ADD Admin
        {
            string username = textBox2.Text.Trim();
            string password = textBox3.Text.Trim();

            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
            {
                MessageBox.Show("Please enter both username and password.", "Missing Data", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            using (SqlConnection conn = new SqlConnection(DB.connectionString))
            {
                string query = "INSERT INTO Admins (Username, Password) VALUES (@Username, @Password)";
                SqlCommand cmd = new SqlCommand(query, conn);
                cmd.Parameters.AddWithValue("@Username", username);
                cmd.Parameters.AddWithValue("@Password", password);

                try
                {
                    conn.Open();
                    cmd.ExecuteNonQuery();
                    MessageBox.Show("Admin added successfully.", "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    conn.Close();

                    // تحديث الداتا جريد
                    LoadAdmins();
                }
                catch (SqlException ex)
                {
                    if (ex.Number == 2627) // UNIQUE constraint error
                        MessageBox.Show("Username already exists.", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    else
                        MessageBox.Show("Error: " + ex.Message);
                }
            }

        }

        private void button7_Click(object sender, EventArgs e) //CLEAR (ADMIN SECTION)
        {
            textBox2.Text = textBox3.Text = textBox4.Text = string.Empty;
        }


        private void button5_Click(object sender, EventArgs e)//REMOVE Admin
        {

            string usernameToDelete = textBox4.Text.Trim();

            if (string.IsNullOrEmpty(usernameToDelete))
            {
                MessageBox.Show("Please enter the username you want to remove.", "Missing Data", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            DialogResult result = MessageBox.Show($"Are you sure you want to delete admin '{usernameToDelete}'?", "Confirm Delete", MessageBoxButtons.YesNo, MessageBoxIcon.Question);

            if (result == DialogResult.Yes)
            {
                using (SqlConnection conn = new SqlConnection(DB.connectionString))
                {
                    string query = "DELETE FROM Admins WHERE Username = @Username";
                    SqlCommand cmd = new SqlCommand(query, conn);
                    cmd.Parameters.AddWithValue("@Username", usernameToDelete);

                    conn.Open();
                    int rowsAffected = cmd.ExecuteNonQuery();
                    conn.Close();

                    if (rowsAffected > 0)
                    {
                        MessageBox.Show("Admin removed successfully.", "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
                        LoadAdmins(); // تحديث الداتا جريد
                    }
                    else
                    {
                        MessageBox.Show("No admin found with that username.", "Not Found", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                    }
                }
            }

        }

        private void button1_Click(object sender, EventArgs e)//EDIT Password (ADMIN SECTION)
        {

            panel_Verify.Visible = true;

        }

        private void button3_Click(object sender, EventArgs e)//CANCEL Button (ADMIN SECTION)
        {
            panel_Verify.Visible = false;
        }

        private void button_Verify_Update_Click(object sender, EventArgs e)//UPDATE Password Button (ADMIN SECTION)
        {
          
        }


        private void panel_Admin_Paint(object sender, PaintEventArgs e)//Admin list
        {
           

            //dataGrid_Admin_List.DataSource = y.ToList();
        }
        private void AddRemoveAdmin_Load(object sender, EventArgs e)
        {
            LoadAdmins();
        }

        private void LoadAdmins()
        {
            using (SqlConnection conn = new SqlConnection(DB.connectionString))
            {
                conn.Open();
                string query = "SELECT Username, Password FROM Admins";

                SqlDataAdapter da = new SqlDataAdapter(query, conn);
                DataTable dt = new DataTable();
                da.Fill(dt);
                dataGrid_Admin_List.DataSource = dt;
            }

        }


        //-----------------------------------END OF ADD/REMOVE ADMIN-----------------------------------------



        //-------------------------------------------OTHER---------------------------------------------------


        private void bunifuFlatButton1_Click(object sender, EventArgs e)
        {
            LoginForm lg = new LoginForm();
            this.Hide();
            lg.Show();
        }

        private void label2_Click(object sender, EventArgs e)
        {

        }

        private void headerPanel_Paint(object sender, PaintEventArgs e)
        {

        }

        private void groupBox_Bill_Enter(object sender, EventArgs e)
        {

        }

        private void button12_Click(object sender, EventArgs e)
        {

        }

        private void sidePanel_Paint(object sender, PaintEventArgs e)
        {

        }

        private void logoPanel_Paint(object sender, PaintEventArgs e)
        {

        }

        private void label31_Click(object sender, EventArgs e)
        {

        }

        private void dataGridView_Bill_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {

        }

        private void button_New_Members_Click(object sender, EventArgs e)
        {
            panel_AddMem.BringToFront(); 
            panel_AddMem.Visible = true; 
            panel_Purchase_Product.Visible = true; 
            panel_Purchase_Product.BringToFront();


        }

        private void button_New_Members_Click_1(object sender, EventArgs e)
        {
            panel_Home.Visible = false;
            panel_AddMem.Visible = true;
            panel_AddMem.BringToFront();
        }

        private void button11_Click(object sender, EventArgs e)
        {
         
            panel_Home.Visible = false;
          
            panel_AddMem.Visible = false;
            panel_Purchase_Product.Visible = true; 
            panel_Purchase_Product.BringToFront();
        }

        private void button11_Click_1(object sender, EventArgs e)
        {
           
            panel_Home.Visible = false;
            panel_AddMem.Visible = false;
            panel_Purchase_Product.Visible = true; 
            panel_Purchase_Product.BringToFront(); // نجيبها قدام
        }

        private void panel_AddMem_Paint(object sender, PaintEventArgs e)
        {

        }

        private void textName_TextChanged(object sender, EventArgs e)
        {

        }

        private void button15_Click(object sender, EventArgs e)
        {
            panel_Home.Visible = false;
            panel_Mem_Info.Visible = true;
            panel_Mem_Info.BringToFront();
        }

        private void panel_Home_Paint(object sender, PaintEventArgs e)
        {

        }

        private void comboFees_SelectedIndexChanged(object sender, EventArgs e)
        {

        }

        private void Price_Lable_TextChanged(object sender, EventArgs e)
        {

        }

        private void dateTimePickerStart_ValueChanged(object sender, EventArgs e)
        {

        }

        private void label38_Click(object sender, EventArgs e)
        {

        }

        private void panel_Admin_AutoSizeChanged(object sender, EventArgs e)
        {

        }

        private void buttonRefreshAdmins_Click(object sender, EventArgs e)
        {
            LoadAdmins();
        }

        private void dataGridView1_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
           
        }

        private void label40_Click(object sender, EventArgs e)
        {

        }

        private void panel_Mem_Info_Paint(object sender, PaintEventArgs e)
        {

        }

        private void label18_Click(object sender, EventArgs e)
        {

        }

        private void textBox_Up_Age_TextChanged(object sender, EventArgs e)
        {

        }

        private void textBox_Up_Weight_TextChanged(object sender, EventArgs e)
        {

        }

        private void comboBox_Up_batch_SelectedIndexChanged(object sender, EventArgs e)
        {

        }

        private void textBox_Id_TextChanged(object sender, EventArgs e)
        {

        }

        private void button9_Click_1(object sender, EventArgs e)
        {
            text_ID.Text = "";
            box_name.Text = "";
            textBox_Rate.Text = "";
            text_Quantity.Text = "";

            text_ID.Focus();
        }

        private void bunifuFlatButton4_Click(object sender, EventArgs e)
        {
            groupBox_Bill.Visible = false;
            panel_Purchase_Product.Visible = true;
        }

        private void text_Pd_Quantity_TextChanged(object sender, EventArgs e)
        {

        }

        private void dataGrid_Purchase_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
            if (e.RowIndex >= 0)
            {
                DataGridViewRow row = dataGrid_Purchase.Rows[e.RowIndex];
                textBox_Pd_Id.Text = row.Cells["ProductID"].Value.ToString();
                text_Pd_Name.Text = row.Cells["ProductName"].Value.ToString();
                text_Pd_Quantity.Text = row.Cells["Quantity"].Value.ToString();
                text_Pd_Price.Text = row.Cells["Rate"].Value.ToString();
                textBox6.Text = row.Cells["Cost"].Value.ToString(); // amount
            }
        }

        private void dataGrid_Admin_List_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {

        }

        private void panel1_Paint(object sender, PaintEventArgs e)
        {

        }

        private void label41_Click(object sender, EventArgs e)
        {

        }

        private void lblTotalMembers_Click(object sender, EventArgs e)
        {

        }

        private void panel_Update_Member_Paint(object sender, PaintEventArgs e)
        {

        }

        private void box_name_TextChanged(object sender, EventArgs e)
        {

        }

        private void label8_Click(object sender, EventArgs e)
        {

        }

        private void bunifuFlatButton1_Click_1(object sender, EventArgs e)
        {
            panel1.Visible = false;
        }

        private void button13_Click(object sender, EventArgs e)
        {
            panel1.Visible = true;
        }

    }

}

```