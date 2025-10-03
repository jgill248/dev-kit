# MUI Date Picker

## Description
Guide for implementing date and date-time pickers using Material-UI DatePicker components with proper validation and formatting.

## Style Rules
- Use @mui/x-date-pickers for date/time selection
- Wrap with LocalizationProvider (dayjs, date-fns, or luxon)
- Implement proper TypeScript types for date values
- Handle null/undefined dates appropriately
- Add validation for date ranges and constraints
- Use controlled components with state management
- Provide accessible labels and helper text
- Handle timezone considerations

## Example: Basic Date Picker

```typescript
import React, { useState } from 'react';
import { LocalizationProvider } from '@mui/x-date-pickers/LocalizationProvider';
import { DatePicker } from '@mui/x-date-pickers/DatePicker';
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs';
import { TextField } from '@mui/material';
import dayjs, { Dayjs } from 'dayjs';

interface DatePickerFieldProps {
  label: string;
  value: Dayjs | null;
  onChange: (date: Dayjs | null) => void;
  minDate?: Dayjs;
  maxDate?: Dayjs;
  disabled?: boolean;
  required?: boolean;
  helperText?: string;
  error?: boolean;
}

export const DatePickerField: React.FC<DatePickerFieldProps> = ({
  label,
  value,
  onChange,
  minDate,
  maxDate,
  disabled = false,
  required = false,
  helperText,
  error = false,
}) => {
  return (
    <LocalizationProvider dateAdapter={AdapterDayjs}>
      <DatePicker
        label={label}
        value={value}
        onChange={onChange}
        minDate={minDate}
        maxDate={maxDate}
        disabled={disabled}
        slotProps={{
          textField: {
            required,
            helperText,
            error,
            fullWidth: true,
          },
        }}
      />
    </LocalizationProvider>
  );
};
```

## Example: Date Range Picker

```typescript
import React, { useState } from 'react';
import { LocalizationProvider } from '@mui/x-date-pickers/LocalizationProvider';
import { DatePicker } from '@mui/x-date-pickers/DatePicker';
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs';
import { Box, Typography } from '@mui/material';
import dayjs, { Dayjs } from 'dayjs';

interface DateRangePickerProps {
  startDate: Dayjs | null;
  endDate: Dayjs | null;
  onStartDateChange: (date: Dayjs | null) => void;
  onEndDateChange: (date: Dayjs | null) => void;
}

export const DateRangePicker: React.FC<DateRangePickerProps> = ({
  startDate,
  endDate,
  onStartDateChange,
  onEndDateChange,
}) => {
  const [error, setError] = useState<string>('');

  const handleStartDateChange = (date: Dayjs | null) => {
    if (date && endDate && date.isAfter(endDate)) {
      setError('Start date must be before end date');
    } else {
      setError('');
      onStartDateChange(date);
    }
  };

  const handleEndDateChange = (date: Dayjs | null) => {
    if (date && startDate && date.isBefore(startDate)) {
      setError('End date must be after start date');
    } else {
      setError('');
      onEndDateChange(date);
    }
  };

  return (
    <LocalizationProvider dateAdapter={AdapterDayjs}>
      <Box sx={{ display: 'flex', gap: 2, flexDirection: 'column' }}>
        <DatePicker
          label="Start Date"
          value={startDate}
          onChange={handleStartDateChange}
          maxDate={endDate || undefined}
          slotProps={{
            textField: {
              error: !!error,
              fullWidth: true,
            },
          }}
        />
        <DatePicker
          label="End Date"
          value={endDate}
          onChange={handleEndDateChange}
          minDate={startDate || undefined}
          slotProps={{
            textField: {
              error: !!error,
              helperText: error,
              fullWidth: true,
            },
          }}
        />
      </Box>
    </LocalizationProvider>
  );
};
```

## Usage

```typescript
import { useState } from 'react';
import { DatePickerField } from './components/DatePickerField';
import { DateRangePicker } from './components/DateRangePicker';
import dayjs, { Dayjs } from 'dayjs';

function App() {
  const [selectedDate, setSelectedDate] = useState<Dayjs | null>(dayjs());
  const [startDate, setStartDate] = useState<Dayjs | null>(null);
  const [endDate, setEndDate] = useState<Dayjs | null>(null);

  return (
    <div>
      <DatePickerField
        label="Select Date"
        value={selectedDate}
        onChange={setSelectedDate}
        minDate={dayjs()}
        required
      />

      <DateRangePicker
        startDate={startDate}
        endDate={endDate}
        onStartDateChange={setStartDate}
        onEndDateChange={setEndDate}
      />
    </div>
  );
}
```
