---
layout: page
title: Timetable
permalink: /magpie22/timetable
---


<link href='https://cdn.jsdelivr.net/npm/fullcalendar@5.3.1/main.min.css' rel='stylesheet' />
<script src='https://cdn.jsdelivr.net/npm/fullcalendar@5.3.1/main.min.js'></script>


<script>

  document.addEventListener('DOMContentLoaded', function() {
    var initialTimeZone = 'local';
    var timeZoneSelectorEl = document.getElementById('timezone-selector');
    var loadingEl = document.getElementById('loading');
    var calendarEl = document.getElementById('calendar');

    var calendar = new FullCalendar.Calendar(calendarEl, {
      timeZone: initialTimeZone,
      contentHeight: 700,
      initialView: 'listYear',
      initialDate: '2022-03-04',
      headerToolbar: '',
      navLinks: true, // can click day/week names to navigate views
      editable: false,
      selectable: true,
      eventLimit: true,
      events: "../../../../calendar-data/index.htm",
      eventTimeFormat: { hour: 'numeric', minute: '2-digit', timeZoneName: 'short' },
    });

    calendar.render();

    // when the timezone selector changes, dynamically change the calendar option
    timeZoneSelectorEl.addEventListener('change', function() {
      calendar.setOption('timeZone', this.value);
    });
  });

</script>

The **preliminary** workshop schedule is now available. Please be aware that while the overall dates have been settled, the composition and timing of sessions might be still subject to change. Please do not forget to [register] in case you want to attend the workshop.

<div style="padding:0 0 15px 0;">
    Timezone:
    <select id='timezone-selector'>
      <option value='local'>local</option>
      <option value='UTC'>UTC</option>
    </select>
</div>


<div id="calendar"></div>

[register]:register
