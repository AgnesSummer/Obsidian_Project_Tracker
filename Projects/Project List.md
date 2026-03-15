#project-dashboard #Work #homepage
### Active

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
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1)
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

  const colTemplate = `${maxNameWidth}px 80px 60px 60px 2fr`
  const container = dv.el('div', '', { attr: { style: 'width: 100%;' } })

  // 表头
  const header = container.createEl('div', {
    attr: {
      style: `
        display: grid;
        grid-template-columns: ${colTemplate};
        justify-items: center;
        gap: 8px;
        padding: 2px 12px 6px;
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
          gap: 8px;
          align-items: center;
          padding: 6px 12px;
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

### Hold

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
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1)
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
  .filter(p => p.projects?.[0]?.status === 'hold')
  .sort(p => ({ high: 3, medium: 2, low: 1 }[p.projects?.[0]?.priority] || 0), 'desc')
  .values

if (pages.length === 0) {
  dv.el('div', '暂无暂停的项目', { attr: { style: 'color: #999; font-size: 13px; padding: 12px;' } })
} else {
  const maxNameWidth = Math.max(
    ...pages.map(p => measureTextWidth(p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '')))
  ) + 10 + 8 + 16

  const colTemplate = `${maxNameWidth}px 80px 60px 60px 2fr`
  const container = dv.el('div', '', { attr: { style: 'width: 100%;' } })

  // 表头
  const header = container.createEl('div', {
    attr: {
      style: `
        display: grid;
        grid-template-columns: ${colTemplate};
        justify-items: center;
        gap: 8px;
        padding: 2px 12px 6px;
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
          gap: 8px;
          align-items: center;
          padding: 6px 12px;
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

### Finish

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
  return dv.pages('#project-info')
    .filter(p => p.file.path.indexOf('Templates') === -1)
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
  .filter(p => p.projects?.[0]?.status === 'finish')
  .sort(p => ({ high: 3, medium: 2, low: 1 }[p.projects?.[0]?.priority] || 0), 'desc')
  .values

if (pages.length === 0) {
  dv.el('div', '暂无已完成的项目', { attr: { style: 'color: #999; font-size: 13px; padding: 12px;' } })
} else {
  const maxNameWidth = Math.max(
    ...pages.map(p => measureTextWidth(p.projects?.[0]?.name || p.file.name.replace(/^Project-/, '')))
  ) + 10 + 8 + 16

  const colTemplate = `${maxNameWidth}px 80px 60px 60px 2fr`
  const container = dv.el('div', '', { attr: { style: 'width: 100%;' } })

  // 表头
  const header = container.createEl('div', {
    attr: {
      style: `
        display: grid;
        grid-template-columns: ${colTemplate};
        justify-items: center;
        gap: 8px;
        padding: 2px 12px 6px;
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
          gap: 8px;
          align-items: center;
          padding: 6px 12px;
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
