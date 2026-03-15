### Project Tracker
```dataviewjs
const projColors = [
  '#7BA7BC','#DDA3A2', '#9B8AA6', '#E7BA86', '#A3C4A8',
  '#9FAEC2', '#4384A3', '#C47A8A', '#7B6B8A', '#C4B87A',
]

const priorityConfig = {
  high: { color: '#6B9E8A', label: 'High' },
  medium: { color: '#9BBDB5', label: 'Medium' },
  low: { color: '#C4D4D0', label: 'Low' },
}

function getSortedProjects() {
  // 按 ctime 排序，确保颜色稳定
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1)
    .filter(p => p.projects?.[0]?.status === 'active')
    .sort(p => p.file.ctime)
    .map(p => p.projects?.[0]?.name || p.file.name.replace(/^Project-/, ''))
    .values
}

function getProjColor(name, sortedProjects) {
  const idx = sortedProjects.indexOf(name)
  return projColors[idx % projColors.length]
}

function measureTextWidth(text, font = '13px sans-serif') {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  ctx.font = font
  return ctx.measureText(text).width
}

const sortedProjects = getSortedProjects()

const pages = dv.pages('#project-info')
  .filter(p => p.file.path.indexOf('Templates') === -1)
  .filter(p => p.projects?.[0]?.status === 'active')
  .sort(p => ({ high: 3, medium: 2, low: 1 }[p.projects?.[0]?.priority] || 0), 'desc')
  .values

if (pages.length === 0) {
  dv.el('div', '暂无进行中的项目', { attr: { style: 'color: #999; font-size: 13px; padding: 12px;' } })
} else {
  const maxNameWidth = Math.max(
    ...pages.map(p => measureTextWidth(p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '')))
  ) + 10 + 8 + 16

  const colTemplate = `${maxNameWidth}px 70px 45px 45px 2fr`
  const container = dv.el('div', '', { attr: { style: 'width: 100%;' } })

  // 表头
  const header = container.createEl('div', {
    attr: {
      style: `
        display: grid;
        grid-template-columns: ${colTemplate};
        justify-items: center;
        gap: 4px;
        padding: 2px 8px 6px;
        font-size: 11px;
        font-weight: 600;
        color: #888;
        letter-spacing: 0.08em;
        text-transform: uppercase;
        border-bottom: 1px solid #d8d8d8;
        margin-bottom: 2px;
      `
    }
  })

  for (let col of ['Project', 'Priority', 'Todo', 'Done', 'Progress']) {
    header.createEl('span', { text: col, attr: { style: 'display: block; width: 100%; text-align: center;' } })
  }

  // 数据行
  for (let p of pages) {
    const tasks = p.file.tasks
    const done = tasks.filter(t => t.completed).length
    const total = tasks.length
    const todo = total - done
    const progress = total === 0 ? 0 : Math.round(done / total * 100)
    const projName = p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '')
    const projColor = getProjColor(projName, sortedProjects)
    const pri = priorityConfig[p.projects?.[0]?.priority] || { color: '#ccc', label: p.projects?.[0]?.priority }

    const row = container.createEl('div', {
      attr: {
        style: `
          display: grid;
          grid-template-columns: ${colTemplate};
          justify-items: center;
          gap: 4px;
          align-items: center;
          padding: 6px 8px;
          border-radius: 8px;
          transition: background 0.2s;
        `
      }
    })

    row.addEventListener('mouseover', () => row.style.background = '#f7f7f7')
    row.addEventListener('mouseout', () => row.style.background = 'transparent')

    const nameEl = row.createEl('div', { attr: { style: 'display: flex; align-items: center; min-width: 0;' } })
    nameEl.createEl('a', {
      text: projName,
      attr: {
        href: p.file.path,
        class: 'internal-link',
        style: `
          display: inline-block;
          background: ${projColor};
          color: #fff;
          font-size: 12px;
          padding: 4px 12px;
          border-radius: 8px;
          text-decoration: none;
          white-space: nowrap;
          text-align: center;
          min-width: ${maxNameWidth - 20}px;
        `
      }
    })

    row.createEl('span', {
      text: pri.label,
      attr: {
        style: `
          background: ${pri.color};
          color: #fff;
          font-size: 11px;
          padding: 4px 8px;
          border-radius: 6px;
          text-align: center;
          display: inline-block;
          width: 60px;
        `
      }
    })

    row.createEl('span', { text: `${todo}`, attr: { style: 'font-size: 13px; color: #888; text-align: center;' } })
    row.createEl('span', { text: `${done}`, attr: { style: 'font-size: 13px; color: #888; text-align: center;' } })

    const barWrap = row.createEl('div', {
      attr: {
        style: `
          display: flex;
          align-items: center;
          gap: 8px;
          justify-self: stretch;
          width: 100%;
        `
      }
    })

    const barBg = barWrap.createEl('div', {
      attr: {
        style: `
          flex: 1;
          height: 6px;
          border-radius: 99px;
          background: #E7E9DD;
          overflow: hidden;
        `
      }
    })

    barBg.createEl('div', {
      attr: {
        style: `
          width: ${progress}%;
          height: 100%;
          border-radius: 99px;
          background: ${projColor};
          transition: width 0.3s;
        `
      }
    })

    barWrap.createEl('span', { text: `${progress}%`, attr: { style: 'font-size: 11px; color: #999; width: 32px;' } })
  }
}
```
### Time Tracker
*✦ Daily: study 1h · work 6h*
```dataviewjs
// Config
const CONFIG = {
  //存时间记录的文件名
  recordFile: `time-records-${moment().format('YYYYMM')}.md`,
  DAILY_HOURS: 7, //一天预计工作7h
  quickAddHours: 0.5, //点一次增加0.5h
  viewType: "month", //week=周视图，month=月视图
};

const projColors = [
  '#7BA7BC','#DDA3A2', '#9B8AA6', '#E7BA86', '#A3C4A8',
  '#9FAEC2', '#4384A3', '#C47A8A', '#7B6B8A', '#C4B87A',
];

// Utils
function getWorkdays(start, end) {
  let count = 0;
  let d = moment(start);
  const endDate = moment(end);
  while (d.isSameOrBefore(endDate)) {
    const day = d.day();
    if (day !== 0 && day !== 6) count++;
    d.add(1, 'day');
  }
  return count;
}

function getViewDateRange(type) {
  const today = moment().format("YYYY-MM-DD");
  const monthStart = moment().startOf('month').format("YYYY-MM-DD");
  const weekStart = moment().startOf('isoWeek').format("YYYY-MM-DD");
  if (type === "week") {
    const weekEnd = moment(weekStart).add(6, 'day').format("YYYY-MM-DD");
    return { start: weekStart, end: weekEnd, today };
  } else {
    return { start: monthStart, end: today, today };
  }
}

// Data loading - 统一查询函数，避免重复
function getProjectPages() {
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1);
}

function getSortedProjects() {
  // 按 ctime 排序，确保颜色稳定（与 Project Tracker 保持一致）
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1)
    .filter(p => p.projects?.[0]?.status === 'active')
    .sort(p => p.file.ctime)
    .map(p => p.projects?.[0]?.name || p.file.name.replace(/^Project-/, ''))
    .values;
}

function loadActiveProjects() {
  return getProjectPages()
    .filter(p => p.projects?.[0]?.status === 'active' && p.projects?.[0]?.priority === 'high')
    .sort(p => p.file.ctime)
    .values;
}

async function loadTimeRecords(config, dateRange, activeProjects) {
  const stats = {};
  // activeProjects 是 values 数组，每个元素是 page 对象
  for (let p of activeProjects) {
    const projName = p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '');
    stats[projName] = { today: 0, week: 0, month: 0 };
  }

  // 如果没有活跃项目，直接返回空结果
  if (activeProjects.length === 0) {
    return { stats: {}, dailyStats: {}, monthTotal: 0 };
  }

  let monthTotal = 0;
  let dailyStats = {};

  if (config.viewType === "week") {
    for (let i = 0; i < 7; i++) {
      const date = moment(dateRange.start).add(i, 'day').format("YYYY-MM-DD");
      dailyStats[date] = {};
    }
  } else {
    let d = moment(dateRange.start);
    while (d.isSameOrBefore(dateRange.today)) {
      dailyStats[d.format("YYYY-MM-DD")] = {};
      d.add(1, 'day');
    }
  }

  let content = '';
  try {
    content = await dv.io.load(config.recordFile);
  } catch (e) {
    // dv.io.load 在文件不存在时会抛出异常
  }

  // 如果内容为空或undefined，创建新文件
  if (!content) {
    const monthTitle = moment().format('YYYY年M月');
    content = `# Time Records ${monthTitle}\n\nFormat: Date | Project | Hours\n\n---\n`;
    try {
      await app.vault.create(config.recordFile, content);
      new Notice(`Created ${config.recordFile}`);
    } catch (createErr) {
      // 文件可能已存在，忽略错误
      console.log('File creation result:', createErr?.message || 'unknown');
    }
  }

  const lines = content.split("\n");
  const monthStart = moment().startOf('month').format("YYYY-MM-DD");
  // 使用 dateRange.start 作为周起始，确保与 dailyStats 初始化一致
  const weekStart = dateRange.start;

  for (let line of lines) {
    const m = line.match(/^(\d{4}-\d{2}-\d{2})\s*\|\s*(.+?)\s*\|\s*(.+)$/);
    if (!m) continue;
    const date = m[1];
    const proj = m[2].trim();
    const h = parseFloat(m[3].replace('h', '')) || 0;
    if (stats[proj]) {
      if (date === dateRange.today) stats[proj].today += h;
      if (date >= weekStart) stats[proj].week += h;
      if (date >= monthStart) stats[proj].month += h;
    }
    if (date >= monthStart) monthTotal += h;
    if (dailyStats[date]) {
      if (!dailyStats[date][proj]) dailyStats[date][proj] = 0;
      dailyStats[date][proj] += h;
    }
  }
  return { stats, dailyStats, monthTotal };
}

// Color management
function getProjColor(name, sortedProjects) {
  const idx = sortedProjects.indexOf(name);
  return projColors[idx % projColors.length];
}

function sortProjects(projs, projList, hoursMap) {
  return [...projs].sort((a, b) => {
    const aIdx = projList.indexOf(a);
    const bIdx = projList.indexOf(b);
    if (aIdx >= 0 && bIdx >= 0) return aIdx - bIdx;
    if (aIdx >= 0) return -1;
    if (bIdx >= 0) return 1;
    return (hoursMap[b] || 0) - (hoursMap[a] || 0);
  });
}

// Render functions
function renderStyles() {
  return `<style>
    .time-row{display:flex;align-items:center;gap:12px;margin:8px 0;}
    .time-cards{display:flex;gap:8px;flex-wrap:wrap;}
    .time-card{background:#f5f5f5;border-radius:8px;padding:5px 10px;width:85px;text-align:center;cursor:pointer;transition:transform 0.1s,box-shadow 0.1s;}
    .time-card:hover{transform:translateY(-2px);box-shadow:0 2px 8px rgba(0,0,0,0.1);}
    .time-card:active{transform:scale(0.96);}
    .time-card-title{font-size:0.65em;color:#888;margin-bottom:1px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;}
    .time-card-value{font-size:0.95em;font-weight:600;color:#4384A3;}
    .time-card-sub{font-size:0.6em;color:#aaa;}
    .time-chart-wrap{display:flex;align-items:center;gap:8px;margin-left:auto;}
    .time-chart-box{position:relative;width:80px;height:80px;}
    .time-chart-center{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);text-align:center;}
    .time-chart-value{font-size:1em;font-weight:600;color:#4384A3;}
    .time-chart-label{font-size:0.55em;color:#888;}
    .time-legend{display:flex;flex-direction:column;gap:3px;}
    .time-legend-item{display:flex;align-items:center;gap:4px;font-size:0.65em;color:#666;white-space:nowrap;}
    .time-legend-dot{width:10px;height:10px;border-radius:2px;flex-shrink:0;}
    .time-legend-pct{margin-left:auto;font-weight:500;color:#4384A3;}
    .chart-divider{border-top:1px solid #ddd;margin-top:20px;padding-top:12px;}
    .chart-title{font-size:0.85em;color:#888;font-weight:500;margin-bottom:10px;}
    .chart-container{display:flex;gap:4px;padding:0 0 8px 0;flex-wrap:wrap;}
    .chart-day{display:flex;flex-direction:column;align-items:center;gap:2px;min-width:24px;flex:1;}
    .chart-date{font-size:0.6em;color:#888;font-weight:500;}
    .chart-bar-wrap{width:100%;height:10px;background:#eee;border-radius:5px;overflow:hidden;}
    .chart-total{font-size:0.5em;color:#666;margin-top:1px;}
    .chart-day.today .chart-date{color:#4384A3;font-weight:600;}
    .chart-day.today{background:#D9E4ED;border-radius:3px;padding:2px;}
  </style>`;
}

function renderCards(projList, stats, getColor, cardsId) {
  let html = `<div class='time-row'><div class='time-cards' id='${cardsId}'>`;
  for (let i = 0; i < projList.length; i++) {
    const proj = projList[i];
    const color = getColor(proj);
    html += `<div class='time-card' data-proj='${proj}' style="border-left:3px solid ${color};">`;
    html += `<div class='time-card-title'>${proj}</div>`;
    html += `<div class='time-card-value'>${stats[proj].today.toFixed(1)}h</div>`;
    html += `<div class='time-card-sub'>周 ${stats[proj].week.toFixed(1)}h<br>月 ${stats[proj].month.toFixed(1)}h</div>`;
    html += `</div>`;
  }
  return html + "</div>";
}

function renderDonutChart(chartId, projList, stats, getColor, targetHours, untrackedHours, viewType) {
  // 根据 viewType 获取对应的总时长
  const periodTotal = viewType === "week"
    ? projList.reduce((sum, p) => sum + (stats[p]?.week || 0), 0)
    : projList.reduce((sum, p) => sum + (stats[p]?.month || 0), 0);
  const utilization = targetHours > 0 ? Math.min(Math.round((periodTotal / targetHours) * 100), 100) : 0;
  const periodLabel = viewType === "week" ? "周" : "月";
  let html = "<div class='time-chart-wrap'>";
  html += `<div class='time-chart-box'>`;
  html += `<canvas id='${chartId}' width='80' height='80' style='width:80px;height:80px;'></canvas>`;
  html += `<div class='time-chart-center'>`;
  html += `<div class='time-chart-value'>${utilization}%</div>`;
  html += `<div class='time-chart-label'>${periodTotal.toFixed(1)}/${targetHours}h</div>`;
  html += `</div></div>`;
  html += "<div class='time-legend'>";
  for (let i = 0; i < projList.length; i++) {
    const proj = projList[i];
    const projHours = viewType === "week" ? (stats[proj]?.week || 0) : (stats[proj]?.month || 0);
    const pct = periodTotal > 0 ? Math.round(projHours / periodTotal * 100) : 0;
    html += `<div class='time-legend-item'><span class='time-legend-dot' style="background:${getColor(proj)}"></span>${proj}<span class='time-legend-pct'>${pct}%</span></div>`;
  }
  if (untrackedHours > 0 && targetHours > 0) {
    const pct = Math.round(untrackedHours / targetHours * 100);
    html += `<div class='time-legend-item'><span class='time-legend-dot' style="background:#E0E0E0"></span>未记录<span class='time-legend-pct'>${pct}%</span></div>`;
  }
  return html + "</div></div></div>";
}

function renderTimeChart(dailyStats, dateRange, projList, getProjColor, viewType, DAILY_HOURS) {
  const title = viewType === "week" ? "Week View" : "Month View";
  let html = "<div class='chart-divider'>";
  html += `<div class='chart-title'>${title}</div>`;
  const dates = Object.keys(dailyStats).sort();

  if (viewType === "week") {
    html += "<div class='chart-container' style='display:flex;gap:6px;flex-wrap:wrap;'>";
    for (const date of dates) {
      const dayNum = moment(date).date();
      const isToday = date === dateRange.today;
      // 只显示 projList 中的项目（high priority）
      const dayProjs = Object.keys(dailyStats[date]).filter(p => projList.includes(p));
      let dayTotal = 0;
      for (const proj of dayProjs) {
        dayTotal += dailyStats[date][proj] || 0;
      }
      const sortedProjs = sortProjects(dayProjs, projList, dailyStats[date]);
      const hours = sortedProjs.map(p => dailyStats[date][p] || 0);
      html += `<div class='chart-day' style='flex:1;display:flex;flex-direction:column;align-items:center;gap:2px;padding:4px 2px;${isToday ? "background:#D9E4ED;border-radius:4px;" : ""}'>`;
      html += `<div style='font-size:11px;color:${isToday ? "#4384A3;font-weight:600" : "#888"};'>${dayNum}</div>`;
      const canvasId = 'chart-bar-' + date;
      const dataJson = JSON.stringify({ projs: sortedProjs, hours: hours, dailyHours: DAILY_HOURS, today: isToday });
      html += `<canvas id="${canvasId}" width="60" height="16" style="width:60px;height:16px;" data-chart='${dataJson}'></canvas>`;
      html += `<div style='font-size:9px;color:#666;'>${dayTotal > 0 ? dayTotal.toFixed(1) + 'h' : '-'}</div>`;
      html += `</div>`;
    }
    html += "</div>";
  } else if (viewType === "month") {
    // Month view: 传统日历布局（周一到周日）
    // 添加星期表头
    const weekDays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
    html += "<div class='chart-container' style='display:flex;flex-direction:column;gap:4px;'>";
    // 表头
    html += "<div style='display:grid;grid-template-columns:repeat(7,1fr);gap:4px;margin-bottom:4px;'>";
    for (const wd of weekDays) {
      html += `<div style='text-align:center;font-size:9px;color:#888;font-weight:500;'>${wd}</div>`;
    }
    html += "</div>";

    // 计算月份第一天是星期几（0=周日，1=周一，...，6=周六）
    const firstDayOfMonth = moment(dateRange.start).date(1).day();
    // 计算周一的偏移量（周一=0，周日=6）
    const mondayOffset = (firstDayOfMonth + 6) % 7;

    // 日历网格
    html += "<div style='display:grid;grid-template-columns:repeat(7,1fr);gap:4px;'>";

    // 添加空白单元格（填充月初的空白天）
    for (let i = 0; i < mondayOffset; i++) {
      html += `<div class='chart-day' style='display:flex;flex-direction:column;align-items:center;gap:1px;padding:4px 2px;'>`;
      html += `<div style='font-size:10px;color:#ccc;'>-</div>`;
      html += `<div style='height:8px;'></div>`;
      html += `<div style='font-size:8px;color:#eee;'>-</div>`;
      html += `</div>`;
    }

    // 渲染有数据的日期
    for (const date of dates) {
      const dayNum = moment(date).date();
      const isToday = date === dateRange.today;
      // 只显示 projList 中的项目（high priority）
      const dayProjs = Object.keys(dailyStats[date]).filter(p => projList.includes(p));
      let dayTotal = 0;
      for (const proj of dayProjs) {
        dayTotal += dailyStats[date][proj] || 0;
      }
      const sortedProjs = sortProjects(dayProjs, projList, dailyStats[date]);
      const hours = sortedProjs.map(p => dailyStats[date][p] || 0);
      const hasData = dayTotal > 0;
      html += `<div class='chart-day' style='display:flex;flex-direction:column;align-items:center;gap:1px;padding:4px 2px;${isToday ? "background:#D9E4ED;border-radius:4px;" : ""}'>`;
      html += `<div style='font-size:10px;color:${isToday ? "#4384A3;font-weight:600" : "#888"};'>${dayNum}</div>`;
      if (hasData) {
        const canvasId = 'chart-bar-' + date;
        const dataJson = JSON.stringify({ projs: sortedProjs, hours: hours, dailyHours: DAILY_HOURS, today: isToday });
        html += `<canvas id="${canvasId}" width="40" height="12" style="width:40px;height:12px;" data-chart='${dataJson}'></canvas>`;
        html += `<div style='font-size:8px;color:#666;'>${dayTotal.toFixed(1)}h</div>`;
      } else {
        html += `<div style='height:8px;'></div>`;
        html += `<div style='font-size:8px;color:#ccc;'>-</div>`;
      }
      html += `</div>`;
    }
    html += "</div></div>";
  }
  return html + "</div>";
}

// Canvas drawing
function drawCanvasBars(getProjColor) {
  setTimeout(() => {
    const canvases = document.querySelectorAll('canvas[id^="chart-bar-"]');
    canvases.forEach(canvas => {
      const ctx = canvas.getContext('2d');
      const w = canvas.width;
      const h = canvas.height;
      try {
        const data = JSON.parse(canvas.dataset.chart || '{}');
        const projs = data.projs || [];
        const hours = data.hours || [];
        const dailyHours = data.dailyHours || 7;
        const totalHours = hours.reduce((sum, h) => sum + h, 0);
        ctx.fillStyle = '#eee';
        ctx.fillRect(0, 0, w, h);
        let x = 0;
        const scale = totalHours >= dailyHours ? totalHours : dailyHours;
        for (let i = 0; i < projs.length; i++) {
          if (hours[i] > 0) {
            const barW = (hours[i] / scale) * w;
            ctx.fillStyle = getProjColor(projs[i]);
            ctx.fillRect(x, 0, barW, h);
            x += barW;
          }
        }
      } catch (e) {}
    });
  }, 100);
}

function drawDonutChart(chartId, projList, stats, getColor, untrackedHours, viewType) {
  setTimeout(() => {
    const canvas = document.getElementById(chartId);
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const labels = [];
    const data = [];
    const colors = [];
    projList.forEach((proj) => {
      labels.push(proj);
      const projHours = viewType === "week" ? (stats[proj]?.week || 0) : (stats[proj]?.month || 0);
      data.push(projHours);
      colors.push(getColor(proj));
    });
    if (untrackedHours > 0) {
      labels.push('未记录');
      data.push(untrackedHours);
      colors.push('#E0E0E0');
    }
    new Chart(ctx, {
      type: 'doughnut',
      data: { labels: labels, datasets: [{ data: data, backgroundColor: colors, borderWidth: 0 }] },
      options: { responsive: false, maintainAspectRatio: false, cutout: '60%', plugins: { legend: { display: false }, tooltip: { enabled: true } } }
    });
  }, 150);
}

// Event binding
function bindCardClick(cardsId, config, today) {
  const container = document.getElementById(cardsId);
  if (!container) return;

  container.querySelectorAll('.time-card').forEach(card => {
    card.addEventListener('click', async () => {
      const proj = card.getAttribute('data-proj');
      const entry = `\n${today} | ${proj} | ${config.quickAddHours}h`;

      // 每次点击时重新获取文件，确保文件对象存在
      let file = app.vault.getAbstractFileByPath(config.recordFile);

      // 如果文件不存在，先创建
      if (!file) {
        const monthTitle = moment().format('YYYY年M月');
        const content = `# Time Records ${monthTitle}\n\nFormat: Date | Project | Hours\n\n---\n`;
        try {
          file = await app.vault.create(config.recordFile, content);
          new Notice(`Created ${config.recordFile}`);
        } catch (e) {
          new Notice(`Failed to create file: ${e.message}`);
          return;
        }
      }

      try {
        await app.vault.append(file, entry);
        new Notice(`+ ${proj} ${config.quickAddHours}h`);
      } catch (e) {
        new Notice(`Failed to append: ${e.message}`);
      }
    });
  });
}

// Main
async function main() {
  const container = dv.el("div", "");
  const cardsId = 'tm-cards-' + Math.random().toString(36).slice(2);
  const chartId = 'tm-chart-' + Math.random().toString(36).slice(2);
  const dateRange = getViewDateRange(CONFIG.viewType);

  // 并行加载数据
  const [sortedProjects, activeProjects] = await Promise.all([
    getSortedProjects(),
    loadActiveProjects()
  ]);

  const { stats, dailyStats, monthTotal } = await loadTimeRecords(CONFIG, dateRange, activeProjects);
  const projList = Object.keys(stats);

  // 如果没有活跃项目，显示提示
  if (projList.length === 0) {
    container.innerHTML = `<div style="color: #999; font-size: 13px; padding: 12px;">暂无 high priority 的 active 项目</div>`;
    return;
  }

  const getColor = (name) => getProjColor(name, sortedProjects);

  // 根据 viewType 计算目标时长
  let targetHours, untrackedHours;
  if (CONFIG.viewType === "week") {
    // 周视图：本周工作日 * DAILY_HOURS
    const weekWorkdays = getWorkdays(dateRange.start, dateRange.end);
    targetHours = weekWorkdays * CONFIG.DAILY_HOURS;
    const weekTotal = projList.reduce((sum, p) => sum + (stats[p]?.week || 0), 0);
    untrackedHours = Math.max(0, targetHours - weekTotal);
  } else {
    // 月视图：本月工作日 * DAILY_HOURS
    const workdays = getWorkdays(moment().startOf('month').format("YYYY-MM-DD"), dateRange.today);
    targetHours = workdays * CONFIG.DAILY_HOURS;
    untrackedHours = Math.max(0, targetHours - monthTotal);
  }

  let html = renderStyles();
  html += renderCards(projList, stats, getColor, cardsId);
  html += renderDonutChart(chartId, projList, stats, getColor, targetHours, untrackedHours, CONFIG.viewType);
  html += renderTimeChart(dailyStats, dateRange, projList, getColor, CONFIG.viewType, CONFIG.DAILY_HOURS);
  container.innerHTML = html;

  // 加载 Chart.js 并绘制图表
  if (typeof Chart === 'undefined') {
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js';
    document.head.appendChild(script);
    await new Promise(resolve => { script.onload = resolve; script.onerror = resolve; });
  }
  drawDonutChart(chartId, projList, stats, getColor, untrackedHours, CONFIG.viewType);
  drawCanvasBars(getColor);
  bindCardClick(cardsId, CONFIG, dateRange.today);
}
main();
```

---
>[!info] Today
>```dataviewjs
>let projs = dv.pages('#project-info').filter(p => p.file.path.indexOf('Templates') === -1 && p.projects?.[0]?.status === 'active').sort(p => ({high:3,medium:2,low:1}[p.projects?.[0]?.priority]||1), 'desc');
>for (let p of projs) {
> let projName = p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '');
> let t = p.file.tasks?.filter(x => !x.completed && x.text.includes('⏫')) || [];
> if (t.length) { dv.el("div", `<a data-href="${p.file.path}" class="internal-link" style="font-weight:600;font-size:0.95em;color:#4384A3;">${projName}</a>`); dv.taskList(t, false); }
>}
>```


**Today** 


























