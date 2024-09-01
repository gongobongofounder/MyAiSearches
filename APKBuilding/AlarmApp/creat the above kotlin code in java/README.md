Here is the Java version of the Kotlin code provided above. This Java code implements a simple To-Do app with the functionality to add tasks, set custom alarm tones, and choose an alarm frequency.

### `activity_main.xml`

This is the same XML layout as before:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <!-- Clock Symbol Section -->
    <ImageView
        android:id="@+id/clockIcon"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginTop="16dp"
        android:layout_centerHorizontal="true"
        android:src="@drawable/ic_clock" />

    <!-- To Do List Section -->
    <ListView
        android:id="@+id/todoList"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_below="@id/clockIcon"
        android:layout_above="@id/buttonLayout"
        android:layout_weight="1" />

    <!-- Button Layout for Add Task, Set Alarm Tone, Set Frequency -->
    <LinearLayout
        android:id="@+id/buttonLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:orientation="horizontal"
        android:gravity="center_horizontal">

        <Button
            android:id="@+id/btnAddTask"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Add Task" />

        <Button
            android:id="@+id/btnSetAlarmTone"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Set Alarm Tone" />

        <Button
            android:id="@+id/btnSetFrequency"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Set Frequency" />
    </LinearLayout>

</RelativeLayout>
```

### `MainActivity.java`

Below is the Java implementation for the `MainActivity.java`:

```java
package com.example.alarmapp;

import android.app.TimePickerDialog;
import android.content.Intent;
import android.media.RingtoneManager;
import android.net.Uri;
import android.os.Bundle;
import android.provider.Settings;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ListView;

import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private ListView todoListView;
    private Button addTaskButton;
    private Button setAlarmToneButton;
    private Button setFrequencyButton;

    private List<String> todoList = new ArrayList<>();
    private ArrayAdapter<String> adapter;
    private Uri selectedRingtoneUri;
    private String alarmFrequency = "Everyday";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize views
        todoListView = findViewById(R.id.todoList);
        addTaskButton = findViewById(R.id.btnAddTask);
        setAlarmToneButton = findViewById(R.id.btnSetAlarmTone);
        setFrequencyButton = findViewById(R.id.btnSetFrequency);

        // Setup ListView adapter
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, todoList);
        todoListView.setAdapter(adapter);

        // Add Task Button
        addTaskButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Open a dialog to enter task (not implemented here)
                String task = "New Task " + (todoList.size() + 1);
                todoList.add(task);
                adapter.notifyDataSetChanged();
            }
        });

        // Set Alarm Tone Button
        setAlarmToneButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(RingtoneManager.ACTION_RINGTONE_PICKER);
                intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TYPE, RingtoneManager.TYPE_ALARM);
                startActivityForResult(intent, 1);
            }
        });

        // Set Frequency Button
        setFrequencyButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Open a dialog to set frequency (not implemented here)
                alarmFrequency = "Every Monday"; // Example frequency
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 1 && resultCode == RESULT_OK) {
            selectedRingtoneUri = data.getParcelableExtra(RingtoneManager.EXTRA_RINGTONE_PICKED_URI);
            // Save or use the selected ringtone URI
        }
    }
}
```

### Explanation:

- **Clock Symbol:** An `ImageView` in the layout file is used to display a clock icon.
- **To Do List:** A `ListView` is used to show the list of tasks, and an `ArrayAdapter` is used to bind data to the `ListView`.
- **Buttons:**
  - **Add Task:** Adds a new task to the list. You can expand this to include a dialog for entering task details.
  - **Set Alarm Tone:** Opens the ringtone picker for the user to select a custom alarm tone.
  - **Set Frequency:** Opens a dialog (to be implemented) to allow the user to choose the frequency of the alarm (e.g., every day, every Monday).

### Additional Notes:
- **Alarm Management:** Implementing alarm functionality requires using Android's `AlarmManager` and creating custom `BroadcastReceiver` classes to handle alarms. This part isn't covered here but is essential for the full functionality of the app.
- **Data Persistence:** Use a database or shared preferences to save the list of tasks, selected ringtone, and frequency so that they persist across app sessions.
- **Dialogs:** You need to implement dialogs for task input and frequency selection, which can be done using `AlertDialog`.

This Java implementation mirrors the functionality of the original Kotlin code and should work similarly when implemented in an Android project.
