The [`@react-native-community/datetimepicker`](https://github.com/react-native-community/react-native-datetimepicker#react-native-datetimepicker) provides the feature to pick date and time using the native components of Android and iOS.

For iOS, there seems to be no problem in picking up both date and time at the same time ('datetime' mode). But for Android, it is not possible out of the box. So, I thought of writing this.

I'll start from the existing usage instructions for the `datetimepicker`
```javascript
import React, {useState} from 'react';
import {View, Button, Platform} from 'react-native';
import DateTimePicker from '@react-native-community/datetimepicker';

const App = () => {
  const [date, setDate] = useState(new Date());
  const [mode, setMode] = useState('date');
  const [show, setShow] = useState(false);

  const onChange = (event, selectedDate) => {
    const currentDate = selectedDate || date;

    setDate(currentDate);
    setShow(Platform.OS === 'ios' ? true : false);
  };

  const showMode = currentMode => {
    setShow(true);
    setMode(currentMode);
  };

  const showDatepicker = () => {
    showMode('date');
  };

  const showTimepicker = () => {
    showMode('time');
  };

  return (
    <View>
      <View>
        <Button onPress={showDatepicker} title="Show date picker!" />
      </View>
      <View>
        <Button onPress={showTimepicker} title="Show time picker!" />
      </View>
      {show && (
        <DateTimePicker
          testID="dateTimePicker"
          timeZoneOffsetInMinutes={0}
          value={date}
          mode={mode}
          is24Hour={true}
          display="default"
          onChange={onChange}
        />
      )}
    </View>
  );
};

export default App;
```
Let's use a clickable `Text` component showing datetime instead of the buttons. We don't need `showTimePicker()` anymore and thus can be removed.
```javascript
        <TouchableOpacity onPress={showDatePicker}>
          <Text>{date}</Text>
        </TouchableOpacity>
```
Now, we need to modify the `onChange()` method to show up both date and time picker. Before that we need a `time` state similar to `date` state.
```javascript
  const [time, setTime] = useState(new Date());
  //...
  const onChange = (event, selectedValue) => {
    setShow(Platform.OS === 'ios');
    if (mode == 'date') {
      const currentDate = selectedValue || new Date();
      setDate(currentDate);
      setMode('time');
      setShow(Platform.OS !== 'ios'); // to show the picker again in time mode
    } else {
      const selectedTime = selectedValue || new Date();
      setTime(selectedTime);
      setShow(Platform.OS === 'ios');
      setMode('date');
    }
  };
```
Now, we have date and time picker working and updating the respective state values. We just need to merge and format the `date` and `time` state.
```javascript
        <TouchableOpacity onPress={showDatePicker}>
          <Text style={styles.title}>{formatDate(date, time)}</Text>
        </TouchableOpacity>
       
//..

const formatDate = (date, time) => {
  return `${date.getDate()}/${date.getMonth() +
    1}/${date.getFullYear()} ${time.getHours()}:${time.getMinutes()}`;
};
```