
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:

## activity_main.xml:
```
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: SANDHIYA M
Registeration Number :212224220086

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:autofillHints="name"
        android:hint="@string/hint_name"
        android:inputType="textPersonName" />

    <EditText
        android:id="@+id/etAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:importantForAutofill="no"
        android:hint="@string/hint_age"
        android:inputType="number" />

    <EditText
        android:id="@+id/etSalary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:importantForAutofill="no"
        android:hint="@string/hint_salary"
        android:inputType="number" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="?android:attr/buttonBarStyle"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            style="?android:attr/buttonBarButtonStyle"
            android:text="@string/btn_add" />

        <Button
            android:id="@+id/btnUpdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            style="?android:attr/buttonBarButtonStyle"
            android:text="@string/btn_update" />

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            style="?android:attr/buttonBarButtonStyle"
            android:text="@string/btn_delete" />
    </LinearLayout>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>


```
## MainActivity.java:
```
package com.example.pmd_ex1;

import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.database.*;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etName, etAge, etSalary;
    Button btnAdd, btnUpdate, btnDelete;
    ListView listView;

    DatabaseReference dbRef;

    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();
    ArrayAdapter<String> adapter;

    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etAge = findViewById(R.id.etAge);
        etSalary = findViewById(R.id.etSalary);

        btnAdd = findViewById(R.id.btnAdd);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance()
                .getReference("Employees");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );

        listView.setAdapter(adapter);

        // Read employee details from Firebase
        dbRef.addValueEventListener(new ValueEventListener() {

            @Override
            public void onDataChange(DataSnapshot snapshot) {
                items.clear();
                keys.clear();

                for (DataSnapshot d : snapshot.getChildren()) {
                    Emp e = d.getValue(Emp.class);

                    if (e != null) {
                        items.add(
                                e.name + " | " + e.age + " | " + e.salary
                        );
                        keys.add(d.getKey());
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(DatabaseError error) {
                Toast.makeText(
                        MainActivity.this,
                        "Read failed: " + error.getMessage(),
                        Toast.LENGTH_LONG
                ).show();
            }
        });

        // ADD employee
        btnAdd.setOnClickListener(v -> {

            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty() || salary.isEmpty()) {
                Toast.makeText(
                        this,
                        "Enter all details",
                        Toast.LENGTH_SHORT
                ).show();
                return;
            }

            Emp e = new Emp(name, age, salary);

            btnAdd.setEnabled(false);

            dbRef.push().setValue(e)
                    .addOnSuccessListener(unused -> {
                        Toast.makeText(
                                this,
                                "Employee added successfully",
                                Toast.LENGTH_SHORT
                        ).show();

                        clearFields();
                    })
                    .addOnFailureListener(error -> {
                        Toast.makeText(
                                this,
                                "Add failed: " + error.getMessage(),
                                Toast.LENGTH_LONG
                        ).show();
                    })
                    .addOnCompleteListener(task ->
                            btnAdd.setEnabled(true));
        });

        // UPDATE selected employee
        btnUpdate.setOnClickListener(v -> {

            if (selectedKey == null) {
                Toast.makeText(
                        this,
                        "Select an employee from the list first",
                        Toast.LENGTH_SHORT
                ).show();
                return;
            }

            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty() || salary.isEmpty()) {
                Toast.makeText(
                        this,
                        "Enter all details",
                        Toast.LENGTH_SHORT
                ).show();
                return;
            }

            Emp e = new Emp(name, age, salary);
            String key = selectedKey;

            dbRef.child(key).setValue(e)
                    .addOnSuccessListener(unused -> {
                        Toast.makeText(
                                this,
                                "Employee updated successfully",
                                Toast.LENGTH_SHORT
                        ).show();

                        clearFields();
                    })
                    .addOnFailureListener(error ->
                            Toast.makeText(
                                    this,
                                    "Update failed: " + error.getMessage(),
                                    Toast.LENGTH_LONG
                            ).show());
        });

        // DELETE selected employee
        btnDelete.setOnClickListener(v -> {

            if (selectedKey == null) {
                Toast.makeText(
                        this,
                        "Select an employee from the list first",
                        Toast.LENGTH_SHORT
                ).show();
                return;
            }

            String key = selectedKey;

            dbRef.child(key).removeValue()
                    .addOnSuccessListener(unused -> {
                        Toast.makeText(
                                this,
                                "Employee deleted successfully",
                                Toast.LENGTH_SHORT
                        ).show();

                        clearFields();
                    })
                    .addOnFailureListener(error ->
                            Toast.makeText(
                                    this,
                                    "Delete failed: " + error.getMessage(),
                                    Toast.LENGTH_LONG
                            ).show());
        });

        // Select employee from the list
        listView.setOnItemClickListener((parent, view, position, id) -> {

            selectedKey = keys.get(position);

            dbRef.child(selectedKey).get()
                    .addOnSuccessListener(data -> {

                        Emp e = data.getValue(Emp.class);

                        if (e != null) {
                            etName.setText(e.name);
                            etAge.setText(e.age);
                            etSalary.setText(e.salary);
                        }
                    })
                    .addOnFailureListener(error ->
                            Toast.makeText(
                                    this,
                                    "Could not load employee: "
                                            + error.getMessage(),
                                    Toast.LENGTH_LONG
                            ).show());
        });
    }

    private void clearFields() {
        etName.setText("");
        etAge.setText("");
        etSalary.setText("");
        selectedKey = null;
        listView.clearChoices();
    }
}

```




## OUTPUT
<img width="1640" height="1012" alt="Screenshot 2026-10-26 115227" src="https://github.com/user-attachments/assets/e5354204-bec1-4cfd-aa78-c88b9026a082" />


<img width="1920" height="1200" alt="Screenshot 2026-10-26 115303" src="https://github.com/user-attachments/assets/2f26961b-c28a-48fc-ac23-1388763f5cb1" />






## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
