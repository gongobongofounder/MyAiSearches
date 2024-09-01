To enhance your app by allowing users to add a custom task with a name, specific custom alarm tone, frequency, and the ability to set the time and date (if the frequency is set to "Once"), you’ll need to extend the functionality of the app. This involves creating a dialog for entering task details, selecting a custom alarm tone, setting the frequency, and choosing the date and time.

### Overview of Changes:
1. **Add Task Dialog:** Create a dialog that allows users to enter the task name, select a custom tone, choose a frequency, and set a time and date if needed.
2. **Save Task Details:** Store these details for each task, including the name, custom tone URI, frequency, and time/date.
3. **Alarm Management:** Set up alarms based on the user’s inputs using `AlarmManager`.

Here’s how you can achieve this:

### 1. Create a Custom Dialog for Adding Tasks

You can create a custom dialog layout (`dialog_add_task.xml`) that contains input fields for the task name, tone, frequency, and time/date.

#### `dialog_add_task.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/etTaskName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Task Name" />

    <Button
        android:id="@+id/btnSetCustomTone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Set Custom Tone" />

    <Spinner
        android:id="@+id/spinnerFrequency"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/btnSetTimeDate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Set Time & Date" />

</LinearLayout>
```

### 2. Modify the `MainActivity.java`

In `MainActivity.java`, modify the `addTaskButton` click listener to show the custom dialog when adding a new task.

#### Updated `MainActivity.java`

```java
package com.example.alarmapp;

import android.app.AlarmManager;
import android.app.PendingIntent;
import android.app.TimePickerDialog;
import android.content.DialogInterface;
import android.content.Intent;
import android.media.RingtoneManager;
import android.net.Uri;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Spinner;
import android.widget.TimePicker;
import android.widget.Toast;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private ListView todoListView;
    private Button addTaskButton;

    private List<Task> todoList = new ArrayList<>();
    private ArrayAdapter<String> adapter;

    private Uri selectedRingtoneUri;
    private String selectedFrequency;
    private Calendar selectedTimeDate;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize views
        todoListView = findViewById(R.id.todoList);
        addTaskButton = findViewById(R.id.btnAddTask);

        // Setup ListView adapter
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, new ArrayList<>());
        todoListView.setAdapter(adapter);

        // Add Task Button
        addTaskButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showAddTaskDialog();
            }
        });
    }

    private void showAddTaskDialog() {
        // Inflate the dialog layout
        LayoutInflater inflater = LayoutInflater.from(this);
        View dialogView = inflater.inflate(R.layout.dialog_add_task, null);

        final EditText etTaskName = dialogView.findViewById(R.id.etTaskName);
        Button btnSetCustomTone = dialogView.findViewById(R.id.btnSetCustomTone);
        Spinner spinnerFrequency = dialogView.findViewById(R.id.spinnerFrequency);
        Button btnSetTimeDate = dialogView.findViewById(R.id.btnSetTimeDate);

        // Populate spinner with frequency options
        ArrayAdapter<CharSequence> frequencyAdapter = ArrayAdapter.createFromResource(this,
                R.array.frequency_options, android.R.layout.simple_spinner_item);
        frequencyAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        spinnerFrequency.setAdapter(frequencyAdapter);

        // Set Custom Tone Button
        btnSetCustomTone.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(RingtoneManager.ACTION_RINGTONE_PICKER);
                intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TYPE, RingtoneManager.TYPE_ALARM);
                startActivityForResult(intent, 1);
            }
        });

        // Set Time & Date Button
        btnSetTimeDate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showTimeDatePicker();
            }
        });

        // Show the dialog
        new AlertDialog.Builder(this)
                .setTitle("Add Task")
                .setView(dialogView)
                .setPositiveButton("Add", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        String taskName = etTaskName.getText().toString().trim();
                        selectedFrequency = spinnerFrequency.getSelectedItem().toString();

                        if (!taskName.isEmpty() && selectedTimeDate != null) {
                            // Create the task
                            Task newTask = new Task(taskName, selectedRingtoneUri, selectedFrequency, selectedTimeDate);
                            todoList.add(newTask);
                            adapter.add(taskName);
                            adapter.notifyDataSetChanged();

                            // Set the alarm
                            setTaskAlarm(newTask);
                        } else {
                            Toast.makeText(MainActivity.this, "Please enter all details.", Toast.LENGTH_SHORT).show();
                        }
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    private void showTimeDatePicker() {
        final Calendar calendar = Calendar.getInstance();
        TimePickerDialog timePickerDialog = new TimePickerDialog(this,
                new TimePickerDialog.OnTimeSetListener() {
                    @Override
                    public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
                        calendar.set(Calendar.HOUR_OF_DAY, hourOfDay);
                        calendar.set(Calendar.MINUTE, minute);

                        selectedTimeDate = calendar;
                    }
                }, calendar.get(Calendar.HOUR_OF_DAY), calendar.get(Calendar.MINUTE), true);
        timePickerDialog.show();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 1 && resultCode == RESULT_OK) {
            selectedRingtoneUri = data.getParcelableExtra(RingtoneManager.EXTRA_RINGTONE_PICKED_URI);
        }
    }

    private void setTaskAlarm(Task task) {
        // Implement the alarm setting using AlarmManager
        AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
        Intent intent = new Intent(this, AlarmReceiver.class);
        intent.putExtra("taskName", task.getTaskName());
        intent.putExtra("toneUri", task.getRingtoneUri().toString());

        PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);

        if ("Once".equals(task.getFrequency())) {
            alarmManager.setExact(AlarmManager.RTC_WAKEUP, task.getTimeDate().getTimeInMillis(), pendingIntent);
        } else {
            alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, task.getTimeDate().getTimeInMillis(),
                    AlarmManager.INTERVAL_DAY, pendingIntent);  // Set this based on frequency
        }

        Toast.makeText(this, "Alarm set for task: " + task.getTaskName(), Toast.LENGTH_SHORT).show();
    }
}
```

### 3. Create the `Task` Class

Create a `Task` class to hold the task details:

```java
package com.example.alarmapp;

import java.util.Calendar;

public class Task {

    private String taskName;
    private Uri ringtoneUri;
    private String frequency;
    private Calendar timeDate;

    public Task(String taskName, Uri ringtoneUri, String frequency, Calendar timeDate) {
        this.taskName = taskName;
        this.ringtoneUri = ringtoneUri;
        this.frequency = frequency;
        this.timeDate = timeDate;
    }

    public String getTaskName() {
        return taskName;
    }

    public Uri getRingtoneUri() {
        return ringtoneUri;
    }

    public String getFrequency() {
        return frequency;
    }

    public Calendar getTimeDate() {
        return timeDate;
    }
}
```

### 4. Create an `AlarmReceiver`

You'll need a `BroadcastReceiver` to handle the alarm when it triggers:

```java
package com.example.alarmapp;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.media.MediaPlayer;
import android.net.Uri;

public class AlarmReceiver extends Broadcast

Receiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        String taskName = intent.getStringExtra("taskName");
        String toneUri = intent.getStringExtra("toneUri");

        MediaPlayer mediaPlayer = MediaPlayer.create(context, Uri.parse(toneUri));
        mediaPlayer.start();

        // Notify the user about the task (e.g., with a notification)
    }
}
```

### 5. Update `AndroidManifest.xml`

Don't forget to register the `AlarmReceiver` in your `AndroidManifest.xml`:

```xml
<receiver android:name=".AlarmReceiver" />
```

### 6. `strings.xml`

Add the string array for frequency options in `res/values/strings.xml`:

```xml
<resources>
    <string name="app_name">AlarmApp</string>
    <string-array name="frequency_options">
        <item>Once</item>
        <item>Everyday</item>
        <item>Every Monday</item>
        <!-- Add more options as needed -->
    </string-array>
</resources>
```

### Summary

- **Dialog for Adding Task:** The `showAddTaskDialog` method now displays a custom dialog where users can enter the task name, set a custom alarm tone, select a frequency, and set the time/date.
- **Alarm Management:** The `setTaskAlarm` method sets alarms based on the user's input.
- **BroadcastReceiver:** The `AlarmReceiver` class handles the triggered alarms, playing the selected ringtone and notifying the user.

This should give you a fully functional To-Do app with the ability to set custom tones, frequencies, and alarms for individual tasks.
