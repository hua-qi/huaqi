---
session_id: agent-4a7eac27
date: 2026-03-23
time_start: 2026-03-23T08:44:27.047Z
time_end: 2026-03-23T08:46:17.632Z
project: users-lianzimeng-working-happy-scripts-toft-guard-demo
git_branch: 
---

[2026-03-23 16:44:27] [user]: 读取 /Users/lianzimeng/working/happy/scripts/toft-guard-demo/node_modules/@toft/react/dist/components/table/demo/ 目录下的所有 .md 文件，返回每个文件的完整内容。文件列表：basic.md, child-table.md, colSpanfixColumn.md, colspan.md, customColumn.md, customEvent.md, customTable.md, drag.md, dynamicTable.md, expand.md, fetchData.md, fixColumn.md, renderSelectionCell.md, resizable.md, selctions.md, size.md, tree.md, virtualTable.md。请返回每个文件的完整内容，不要省略。
[2026-03-23 16:44:36] [assistant]: 我将读取该目录下的所有 .md 文件。让我逐个读取这些文件。
[2026-03-23 16:46:17] [assistant]: 完美！我已成功读取了全部18个 .md 文件。以下是每个文件的完整内容汇总：

## 1. basic.md - 基本使用

```markdown
---
category: 基本组件
order: 0
title: 基本使用
---

### 基本使用

简单表格，最基础的用法
注意：column 的 label 和 children 都是控制表头的文案，children 的优先级比较高

```jsx
import { useState } from 'react';
import { Table } from '@toft/react';
const { Column } = Table;

function Demo() {
  const dataSource = [
    {
      key: '1',
      name: '张三',
      employeeNumber: '43568',
      certificateNo: '230301199710225113',
      department: '研发线-效率工程部/系统研发'
    },
    {
      key: '2',
      name: '李四',
      employeeNumber: '17290',
      certificateNo: '370881199704030715',
      department: '新业务/海外'
    },
    {
      key: '3',
      name: '王五',
      employeeNumber: '17292',
      certificateNo: '513436200012188184',
      department: '审计部/税务审计'
    }
  ];

  return (
    <Table data={dataSource} rowClassName="k-table-row" bordered>
      <Column dataKey="name">姓名</Column>
      <Column dataKey="employeeNumber">员工工号</Column>
      <Column dataKey="department">部门</Column>
      <Column dataKey="certificateNo">证件号码</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 2. child-table.md - 嵌套子表格

```markdown
---
category: 基本组件
order: 11
title: 嵌套子表格
---

### 嵌套子表格

如果需要渲染可以展开的表格，除了需要在 Table 传 expandedRowRender 这个方法外，还必须要指定 rowKey（默认为 key），Table 会根据 rowKey 取得行唯一标识符。
如果 rowKey 为 Function，则会把 rowKey(record) 的结果作为行唯一 ID
如果 rowKey 为 string 类型，则会把 record[rowKey] 作为行唯一 ID

expandedKeys 完全受控 需要组件使用者自己维护

```jsx
import { useState } from 'react';
import { Table } from '@toft/react';
const { Column } = Table;

function Demo() {
  const [expandedKeys, setExpandedKeys] = useState(['1']);

  const onExpand = (expanded, record) => {
    let expandKeys = [...expandedKeys];
    if (expanded) {
      expandKeys.push(record.key);
    } else {
      expandKeys = expandKeys.filter((item) => item !== record.key);
    }
    setExpandedKeys(expandKeys);
  };

  const data = [
    {
      key: '1',
      name: '张三',
      employeeNumber: '43568',
      certificateNo: '230301199710225113',
      department: '研发线-效率工程部/系统研发'
    },
    {
      key: '2',
      name: '李四',
      employeeNumber: '17290',
      certificateNo: '370881199704030715',
      department: '新业务/海外'
    },
    {
      key: '3',
      name: '王五',
      employeeNumber: '17292',
      certificateNo: '513436200012188184',
      department: '审计部/税务审计'
    }
  ];

  const expandedRowRender = (record, index, expanded) => {
    return (
      <Table data={data} rowClassName="k-table-row" style={{ paddingLeft: 30 }}>
        <Column dataKey="name">姓名</Column>
        <Column dataKey="employeeNumber">员工工号</Column>
        <Column dataKey="department">部门</Column>
        <Column dataKey="certificateNo">证件号码</Column>
      </Table>
    );
  };

  return (
    <Table data={data} rowClassName="k-table-row" rowExpansion={{ expandedKeys, onExpand, expandedRowRender }}>
      <Column dataKey="name">姓名</Column>
      <Column dataKey="employeeNumber">员工工号</Column>
      <Column dataKey="department">部门</Column>
      <Column dataKey="certificateNo">证件号码</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 3. colSpanfixColumn.md - 单元格合并并且固定列

```markdown
---
category: 基本组件
order: 3
title: 单元格合并并且固定列
---

### 单元格合并并且固定列

```jsx
import { Table } from '@toft/react';
const { Column } = Table;
const fixedData = [];

for (let i = 0; i < 50; i += 1) {
  fixedData.push({
    key: i,
    name: ['张三', '李四', '王五'][i % 3],
    employeeNumber: (i + 12) * 13,
    certificateNo: '513436200012188184',
    entryDate: '2021-02-28',
    leaveDate: '2019-02-28',
    department: ['研发线-效率工程部/系统研发', '新业务/海外', '审计部/税务审计'][i % 3],
    company: ['北京一笑科技发展有限公司', '北京达佳互联信息技术有限公司', '北京快手广告有限公司'][i % 3]
  });
}

ReactDOM.render(
  <Table scroll={{ x: 2000, y: 500 }} data={fixedData}>
    <Column dataKey="name" fixed="left" width={100}>
      姓名
    </Column>
    <Column dataKey="employeeNumber" fixed="left" width={120}>
      员工工号
    </Column>
    <Column dataKey="company">公司</Column>
    <Column dataKey="department">部门</Column>
    <Column dataKey="certificateNo">证件号码</Column>
    <Column dataKey="entryDate">入职日期</Column>
    <Column
      dataKey="leaveDate"
      fixed="right"
      width={100}
      align="center"
      render={() => {
        return {
          children: '111111',
          props: { colSpan: 2 }
        };
      }}
    >
      离职日期
    </Column>
    <Column
      dataKey="repayment"
      width={100}
      fixed="right"
      render={() => {
        return {
          props: { colSpan: 0 },
          children: 'last1'
        };
      }}
    >
      操作
    </Column>
  </Table>,
  mountNode
);
```
```

## 4. colspan.md - 表头分组以及表格行列合并

```markdown
---
category: 基本组件
order: 1
title: 表头分组以及表格行列合并
---

### 表头分组以及表格行列合并

- 表头可通过设置 column.colSpan 进行表头的列合并
- 表格支持行/列合并，使用 render 里的单元格属性 colSpan 或者 rowSpan 设值为 0 时，设置的表格不会渲染。

```jsx
import { Table } from '@toft/react';
const { Column, ColumnGroup } = Table;

const data = [
  {
    key: '1',
    name: '张三',
    age: 32,
    tel: '0571-22098909',
    phone: 18889898989,
    address: '上地西路-6号',
    workplace: '北京',
    department: '新业务/海外'
  },
  {
    key: '2',
    name: '李四',
    tel: '0571-22098333',
    phone: 18889898888,
    department: '新业务/海外',
    age: 42,
    workplace: '北京',
    address: '马连洼一号院'
  },
  {
    key: '3',
    name: '王五',
    age: 32,
    tel: '0575-22098909',
    phone: 18900010002,
    workplace: '杭州',
    address: '融泽家园 1号楼-1-102',
    department: '审计部/税务审计'
  },
  {
    key: '4',
    name: '赵六',
    age: 18,
    tel: '0575-22098909',
    workplace: '深圳',
    phone: 18900010002,
    address: '融泽家园 12号楼-2-502',
    department: '新业务/海外'
  }
];

const renderContent = (value, row, index) => {
  const obj = {
    children: value,
    props: {}
  };
  if (index === 4) {
    obj.props.colSpan = 0;
  }
  return obj;
};

const renderTel = (value, row, index) => {
  const obj = {
    children: value,
    props: {}
  };
  if (index === 2) {
    obj.props.rowSpan = 2;
  }
  if (index === 3) {
    obj.props.colSpan = 0;
  }

  return obj;
};

const renderName = (text, row, index) => {
  if (index < 4) {
    return <a>{text}</a>;
  }
  return {
    children: <a>{text}</a>,
    props: {
      colSpan: 5
    }
  };
};

ReactDOM.render(
  <Table data={data}>
    <Column dataKey="name">姓名</Column>
    <ColumnGroup style={{ background: 'red'}} className='red' label="基本信息">
      <Column dataKey="age">年龄</Column>
      <Column dataKey="tel" colSpan={2} render={renderTel} align="center">
        联系方式
      </Column>
      <Column dataKey="phone" render={renderContent} colSpan={0}>
        电话
      </Column>
      <Column dataKey="address" render={renderContent}>
        地址
      </Column>
    </ColumnGroup>
    <ColumnGroup label="职位信息">
      <Column
        dataKey="department"
        render={renderContent}
        onCell={() => ({
          colSpan: 2
        })}
      >
        所在部门
      </Column>
      <Column
        dataKey="workplace"
        onCell={() => ({
          colSpan: 0
        })}
      >
        工作地点
      </Column>
    </ColumnGroup>
  </Table>,
  mountNode
);
```
```

## 5. customColumn.md - 自定义单元格渲染

```markdown
---
category: 基本组件
order: 3
title: 自定义单元格渲染
---

### 自定义单元格渲染

可以通过 Column.render 来自定义某一列的渲染

```jsx
import { Table, Icon } from '@toft/react';
const { Column } = Table;

const data = [
  {
    key: '1',
    firstName: 'John',
    lastName: 'Brown',
    age: 32,
    address: 'New York No. 1 Lake Park'
  },
  {
    key: '2',
    firstName: 'Jim',
    lastName: 'Green',
    age: 42,
    address: 'London No. 1 Lake Park'
  },
  {
    key: '3',
    firstName: 'Joe',
    lastName: 'Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park'
  }
];
const renderUserTag = (name, record) => {
  return (
    <div className="k-render-user">
      <span className="k-render-user-tag">
        <Icon type="user"></Icon>
      </span>
      {name}
    </div>
  );
};

ReactDOM.render(
  <Table data={data}>
    <Column dataKey="firstName" width={212} render={renderUserTag}>
      用户
    </Column>
    <Column dataKey="lastName">Last Name</Column>
    <Column dataKey="age">age</Column>
    <Column dataKey="address">address</Column>
  </Table>,
  mountNode
);
```

```css
.k-render-user {
  display: flex;
  align-items: center;
}
.k-render-user-tag {
  width: 30px;
  height: 30px;
  margin-right: 10px;
  font-size: 16px;
  line-height: 30px;
  text-align: center;
  border: 1px solid #333;
  border-radius: 50%;
}
```
```

## 6. customEvent.md - 自定义行或单元格事件以及属性

```markdown
---
category: 基本组件
order: 7
title: 自定义行或单元格事件以及属性
---

### 自定义行或单元格事件以及属性

- 传入 onRow/OnHeaderClick 可以定义表格或表头行的原生事件或属性。
- 传入 column.onCell/column.onHeaderCell 可以定义表格或表头单元格原生事件或属性。

```jsx
import { useMemo } from 'react';
import { Table } from '@toft/react';
const { Column } = Table;

const data = [
  {
    key: '1',
    name: '张三',
    employeeNumber: '43568',
    certificateNo: '230301199710225113',
    department: '研发线-效率工程部/系统研发'
  },
  {
    key: '2',
    name: '李四',
    employeeNumber: '17290',
    certificateNo: '370881199704030715',
    department: '新业务/海外'
  },
  {
    key: '3',
    name: '王五',
    employeeNumber: '17292',
    certificateNo: '513436200012188184',
    department: '审计部/税务审计'
  }
];

function Demo() {
  const handleClickHead = useMemo(
    () => (columns, index) => {
      return {
        onMouseEnter: (e) => console.log('mouse enter: ', columns, index),
        onMouseLeave: (e) => console.log('mouse leave: ', columns, index)
      };
    },
    []
  );
  const handleClickRow = useMemo(
    () => (record, index) => {
      const props = {
        className: 'my-tr-class'
      };

      if (index === 2) {
        return {
          ...props,
          onClick: (e) => console.log('mouse click: ', record, index)
        };
      } else {
        return {
          ...props
        };
      }
    },
    []
  );
  const onCell = (record) => {
    console.log(record);
    return {
      className: 'test-row'
    };
  };

  const onHeaderCell = (record) => {
    return {
      className: 'test-head'
    };
  };
  return (
    <Table data={data} onHeaderClick={handleClickHead} onRow={handleClickRow}>
      <Column dataKey="name" onCell={onCell} onHeaderCell={onHeaderCell}>
        姓名
      </Column>
      <Column dataKey="employeeNumber">员工工号</Column>
      <Column dataKey="department">部门</Column>
      <Column dataKey="certificateNo">证件号码</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 7. customTable.md - 表头吸顶

```markdown
---
category: 基本组件
order: 4
title: 表头吸顶
---

### 表头吸顶

该 demo 主要展示 如何通过自定义表格展示配合 react-sticky 实现表头吸顶效果

**注意：** components.header.wrapper 存在的情况下 会将表头单独渲染为一个 表格

```jsx
import { Table } from '@toft/react';
import { Sticky, StickyContainer } from 'react-sticky';
const { Column } = Table;

const data = [
  {
    key: '1',
    firstName: 'John',
    lastName: 'Brown',
    age: 32,
    address: 'New York No. 1 Lake Park'
  },
  {
    key: '2',
    firstName: 'Jim',
    lastName: 'Green',
    age: 42,
    address: 'London No. 1 Lake Park'
  },
  {
    key: '3',
    firstName: 'Joe',
    lastName: 'Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park'
  }
];

function Wrapper(props) {
  return (
    <Sticky topOffset={0} containerElement="thead" placeholderElement="tr" childrenElement="tr">
      {({ style, isSticky }) => {
        return (
          <div style={{ ...style, top: 56, zIndex: 999, overflow: 'auto' }} className="toft-table-fixed-header">
            <div className="toft-table-header">
              <div className="toft-table-fixed">{props.children}</div>
            </div>
          </div>
        );
      }}
    </Sticky>
  );
}

const components = {
  header: {
    wrapper: Wrapper
  }
};

ReactDOM.render(
  <StickyContainer>
    <Table data={data} components={components} scroll={{ y: true }} bordered>
      <Column dataKey="firstName" width="120px">
        First Name
      </Column>
      <Column dataKey="lastName">Last Name</Column>
      <Column dataKey="age">age</Column>
      <Column dataKey="address">address</Column>
    </Table>
  </StickyContainer>,
  mountNode
);
```
```

## 8. drag.md - 拖拽排序表格

```markdown
---
category: 基本组件
order: 13
title: 拖拽排序表格
---

### 拖拽排序表格

以配合 react-sortable-hoc@2.0.0 实现拖拽排序

```jsx
import { useState } from 'react';
import { Table } from '@toft/react';
import { SortableContainer, SortableElement } from 'react-sortable-hoc';
const { Column } = Table;

const arrayMoveMutate = (array, from, to) => {
  const startIndex = to < 0 ? array.length + to : to;

  if (startIndex >= 0 && startIndex < array.length) {
    const item = array.splice(from, 1)[0];
    array.splice(startIndex, 0, item);
  }
};

const arrayMove = (array, from, to) => {
  array = [...array];
  arrayMoveMutate(array, from, to);
  return array;
};
const columns = [
  {
    title: '姓名',
    dataKey: 'name',
    width: 120
  },
  {
    title: '工号',
    dataKey: 'employeeNumber',
    width: 100
  },
  {
    title: '身份证号',
    dataKey: 'certificateNo',
    width: 180
  },
  {
    title: '部门',
    dataKey: 'department'
  }
];
const initialData = [
  {
    key: '1',
    name: '张三',
    employeeNumber: '43568',
    certificateNo: '230301199710225113',
    department: '研发线-效率工程部/系统研发'
  },
  {
    key: '2',
    name: '李四',
    employeeNumber: '17290',
    certificateNo: '370881199704030715',
    department: '新业务/海外'
  },
  {
    key: '3',
    name: '王五',
    employeeNumber: '17292',
    certificateNo: '513436200012188184',
    department: '审计部/税务审计'
  }
];

function Demo() {
  const [data, setData] = useState(initialData);
  const SortableWrapper = SortableContainer((props) => {
    return <tbody {...props}></tbody>;
  });

  const SortableItem = SortableElement((props) => {
    return (
      <tr
        {...props}
        style={{
          cursor: 'move',
          background: '#fff'
        }}
      />
    );
  });

  function onSortEnd({ oldIndex, newIndex }) {
    console.log('🚀 ~ file: drag.md ~ line 95 ~ onSortEnd ~ oldIndex', oldIndex, newIndex);
    if (oldIndex !== newIndex) {
      const newData = arrayMove([].concat(data), oldIndex, newIndex).filter((el) => !!el);

      setData(newData);
    }
  }
  const DraggableContainer = (props) => {
    return (
      <SortableWrapper
        onSortEnd={onSortEnd}
        helperContainer={() => document.querySelector('.toft-drag-table-container table tbody')}
        updateBeforeSortStart={({ node }) => {
          const tds = node.querySelectorAll('td');
          tds.forEach((td) => {
            td.style.width = td.clientWidth + 'px';
          });
        }}
        {...props}
      />
    );
  };
  const DraggableRow = (props) => {
    const { record, index, ...rest } = props;
    return <SortableItem key={index} index={index} {...rest} />;
  };

  const components = {
    body: {
      wrapper: DraggableContainer,
      row: DraggableRow
    }
  };

  return (
    <Table data={data} components={components} className="toft-drag-table-container">
      {columns.map((item) => (
        <Column key={item.key} {...item}>
          {item.title}
        </Column>
      ))}
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 9. dynamicTable.md - 动态控制表格

```markdown
---
category: 基本组件
order: 9
title: 动态控制表格
---

### 动态控制表格

- 可查看不同属性的组合的效果

```jsx
import React, { useRef, useState, useMemo } from 'react';
import { Table, Switch, Form, Radio } from '@toft/react';
const { Column } = Table;
const FormItem = Form.Item;
const RadioGroup = Radio.Group;

const fixedData = () => {
  const data = [];
  for (let i = 0; i < 10; i += 1) {
    data.push({
      key: i + 1,
      name: ['张三', '李四', '王五'][i % 3],
      employeeNumber: (i + 12) * 13,
      certificateNo: '513436200012188184',
      entryDate: '2021-02-28',
      leaveDate: '2019-02-28',
      department: ['研发线-效率工程部/系统研发', '新业务/海外', '审计部/税务审计'][i % 3],
      company: ['北京一笑科技发展有限公司', '北京达佳互联信息技术有限公司', '北京快手广告有限公司'][i % 3]
    });
  }
  return data;
};

const expandData = [
  {
    key: '1',
    name: '张三',
    employeeNumber: '43568',
    certificateNo: '230301199710225113',
    department: '研发线-效率工程部/系统研发',
    children: [
      {
        key: '1-2',
        name: '张三2',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外',
        children: [
          {
            key: '1-2-1',
            name: '张三3',
            employeeNumber: '17290',
            certificateNo: '370881199704030715',
            department: '新业务/海外'
          }
        ]
      }
    ]
  },
  {
    key: '2',
    name: '李四',
    employeeNumber: '17290',
    certificateNo: '370881199704030715',
    department: '新业务/海外',
    children: [
      {
        key: '2-1',
        name: '李四2',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外',
        children: [
          {
            key: '2-1-1',
            name: '张三3',
            employeeNumber: '17290',
            certificateNo: '370881199704030715',
            department: '新业务/海外'
          }
        ]
      },
      {
        key: '2-2',
        name: '李四3',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外'
      }
    ]
  },
  {
    key: '3',
    name: '王五',
    employeeNumber: '17292',
    certificateNo: '513436200012188184',
    department: '审计部/税务审计'
  }
];

function Demo() {
  const formRef = useRef();

  const [selectedKeys, setSelectedKeys] = useState([]);
  const [expandedKeys, setExpandedKeys] = useState([]);

  const onExpandChange = (expanded, record) => {
    let expandKeys = [...expandedKeys];
    if (expanded) {
      expandKeys.push(record.key);
    } else {
      expandKeys = expandKeys.filter((item) => item !== record.key);
    }
    setExpandedKeys(expandKeys);
  };

  const getData = () => {
    if (formValues.empty) {
      return [];
    } else if (formValues.expandable) {
      return expandData;
    } else {
      return fixedData();
    }
  };

  const rowSelection = {
    selectable: true,
    selectedKeys,
    getCheckboxProps: (r) => ({
      disabled: r.key === '2-2' || r.key === '1-2-1'
    }),
    onSelectChange: (record, selectedKeys) => {
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ selectedKeys', selectedKeys);
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ record', record);
    },
    onSelect: (rowKey, selected, record) => {
      console.log(`rowKey: ${rowKey}`, 'selected: ', selected, record);
      if (selected && rowKey) {
        setSelectedKeys((pre) => [...pre, rowKey]);
      } else {
        setSelectedKeys((pre) => pre.filter((item) => item !== rowKey));
      }
    },
    onSelectAll: (selected, selectedKeys, selectedRows) => {
      console.log(`selected: ${selected}`, 'selectedKeys: ', selectedKeys);
      console.log(`selected selectedRows: ${selected}`, 'selectedRows: ', selectedRows);
      if (selected && selectedKeys) {
        setSelectedKeys(selectedKeys);
      } else {
        setSelectedKeys([]);
      }
    }
  };
  const defaultValue = { size: 'middle', isShowHeader: true, checkbox: true };
  const [formValues, setFormValues] = useState(defaultValue);

  const onFieldValueChange = (formItemKey, formItemValue) => {
    setFormValues((pre) => ({ ...pre, [formItemKey]: formItemValue }));
  };

  const scroll = useMemo(() => {
    const scrollValue = {};
    if (formValues.isFixedHead) {
      scrollValue.y = 200;
    }
    if (formValues.isFixedColumn) {
      scrollValue.x = 1000;
    }
    return scrollValue;
  }, [formValues]);

  return (
    <div>
      <Form ref={formRef} defaultFieldsValue={defaultValue} onFieldValueChange={onFieldValueChange}>
        <FormItem label="固定表头：" formItemKey="isFixedHead" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="固定列：" formItemKey="isFixedColumn" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="展示表头：" formItemKey="isShowHeader" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="可展开表格：" formItemKey="expandable" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="展示边框：" formItemKey="bordered" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="空数据：" formItemKey="empty" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="选择框：" formItemKey="checkbox" span={4}>
          <Switch toFormItem />
        </FormItem>
        <FormItem label="尺寸：" formItemKey="size" span={12}>
          <RadioGroup toFormItem>
            <Radio.Button value="small">small</Radio.Button>
            <Radio.Button value="middle">middle</Radio.Button>
            <Radio.Button value="large">large</Radio.Button>
          </RadioGroup>
        </FormItem>
      </Form>
      <Table
        showHeader={formValues.isShowHeader}
        data={getData()}
        rowSelection={formValues.checkbox ? rowSelection : undefined}
        rowClassName="k-table-row"
        size={formValues.size}
        scroll={scroll}
        bordered={formValues.bordered}
        rowExpansion={
          formValues.expandable
            ? {
                expandedKeys,
                onExpand: onExpandChange
              }
            : undefined
        }
      >
        <Column dataKey="name" fixed={formValues.isFixedColumn ? 'left' : undefined}>
          姓名
        </Column>
        <Column dataKey="employeeNumber">员工工号</Column>
        <Column dataKey="department" width={300}>
          部门
        </Column>
        <Column dataKey="certificateNo">证件号码</Column>
      </Table>
    </div>
  );
}

ReactDOM.render(<Demo />, mountNode);
```
```

## 10. expand.md - 可展开表格

```markdown
---
category: 基本组件
order: 10
title: 可展开表格
---

### 可展开表格

如果需要渲染可以展开的表格，除了需要在 Table 传 expandedRowRender 这个方法外，还必须要指定 rowKey（默认为 key），Table 会根据 rowKey 取得行唯一标识符。
如果 rowKey 为 Function，则会把 rowKey(record) 的结果作为行唯一 ID
如果 rowKey 为 string 类型，则会把 record[rowKey] 作为行唯一 ID

expandedKeys 完全受控 需要组件使用者自己维护

```jsx
import { useState } from 'react';
import { Table } from '@toft/react';
const { Column } = Table;

function Demo() {
  const [expandedKeys, setExpandedKeys] = useState(['1']);

  const onExpand = (expanded, record) => {
    let expandKeys = [...expandedKeys];
    if (expanded) {
      expandKeys.push(record.key);
    } else {
      expandKeys = expandKeys.filter((item) => item !== record.key);
    }
    setExpandedKeys(expandKeys);
  };

  const data = [
    {
      key: '1',
      name: '张三',
      employeeNumber: '43568',
      certificateNo: '230301199710225113',
      department: '研发线-效率工程部/系统研发'
    },
    {
      key: '2',
      name: '李四',
      employeeNumber: '17290',
      certificateNo: '370881199704030715',
      department: '新业务/海外'
    },
    {
      key: '3',
      name: '王五',
      employeeNumber: '17292',
      certificateNo: '513436200012188184',
      department: '审计部/税务审计'
    }
  ];

  const expandedRowRender = (record, index, expanded) => {
    return <div>这里是详情</div>;
  };

  return (
    <Table data={data} rowClassName="k-table-row" rowExpansion={{ expandedKeys, onExpand, expandedRowRender }}>
      <Column dataKey="name">姓名</Column>
      <Column dataKey="employeeNumber">员工工号</Column>
      <Column dataKey="department">部门</Column>
      <Column dataKey="certificateNo">证件号码</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 11. fetchData.md - 模拟接口获取数据

```markdown
---
category: 基本组件
order: 6
title: 模拟接口获取数据
---

### 模拟接口获取数据

```jsx
import { useEffect, useState, useCallback } from 'react';
import { Table, Pagination, Loading, Icon, Space } from '@toft/react';

const defaultPageSize = 5;
const fixedData = [];

for (let i = 0; i < 20; i += 1) {
  fixedData.push({
    key: i,
    name: ['张三', '李四', '王五'][i % 3],
    employeeNumber: (i + 12) * 13,
    certificateNo: '513436200012188184',
    entryDate: '2021-02-28',
    leaveDate: '2019-02-28',
    department: ['研发线-效率工程部/系统研发', '新业务/海外', '审计部/税务审计'][i % 3],
    company: ['北京一笑科技发展有限公司', '北京达佳互联信息技术有限公司', '北京快手广告有限公司'][i % 3],
    child: i === 2 && [
      {
        key: '1-2',
        name: '张三2',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外',
        child: [
          {
            key: '1-2-1',
            name: '张三3',
            employeeNumber: '17290',
            certificateNo: '370881199704030715',
            department: '新业务/海外'
          }
        ]
      }
    ]
  });
}
function Demo() {
  const { Column } = Table;

  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState({ pageSize: 5 });

  const fetchData = useCallback(
    (currentPage = 1, pageSize = 5) => {
      setLoading(true);
      let page = { pageSize, current: currentPage };
      return new Promise((res, rej) => {
        setTimeout(() => {
          let dataSource = fixedData.slice((currentPage - 1) * page.pageSize, currentPage * page.pageSize);
          res(dataSource);
        }, 1500);
      }).then((dataSource) => {
        setLoading(false);
        setPagination(page);
        setData(dataSource);
      });
    },
    [setLoading]
  );

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return (
    <Loading loading={loading}>
      <Table data={data} scroll={{ x: 2000, y: 200 }}>
        <Column dataKey="name" fixed="left" width={100}>
          姓名
        </Column>
        <Column dataKey="employeeNumber" fixed="left" width={120}>
          员工工号
        </Column>
        <Column dataKey="company">公司</Column>
        <Column dataKey="department">部门</Column>
        <Column dataKey="certificateNo">证件号码</Column>
        <Column dataKey="entryDate">入职日期</Column>
        <Column dataKey="leaveDate">离职日期</Column>
        <Column dataKey="repayment" fixed="right" width={140} render={() => <a>查看详情</a>}>
          操作
        </Column>
      </Table>
      <Pagination
        style={{ marginTop: 20 }}
        current={pagination.current}
        total={fixedData.length}
        pageSize={pagination.pageSize}
        onChange={(page, pageSize) => {
          fetchData(page, pageSize);
        }}
        showQuickJumper={true}
      />
    </Loading>
  );
}

ReactDOM.render(<Demo />, mountNode);
```
```

## 12. fixColumn.md - 固定头和列

```markdown
---
category: 基本组件
order: 2
title: 固定头和列
---

### 固定头和列

- 可以通过设置 column 的 fixed 属性以及 scroll.x 来进行列固定，通过设置 scroll.y 来进行表头固定。

若列头与内容不对齐或出现列重复，请指定固定列的宽度 width。如果指定 width 不生效，请尝试建议留一列不设宽度以适应弹性布局，或者检查是否有超长连续字段破坏布局。

建议指定 scroll.x 为大于表格宽度的固定值或百分比。推荐设置为 >=所有固定列宽之和+所有表格列宽之和 的固定数值。

```jsx
import { Table } from '@toft/react';
const { Column } = Table;
const fixedData = [];

for (let i = 0; i < 50; i += 1) {
  fixedData.push({
    key: i,
    name: ['张三', '李四', '王五'][i % 3],
    employeeNumber: (i + 12) * 13,
    certificateNo: '513436200012188184',
    entryDate: '2021-02-28',
    leaveDate: '2019-02-28',
    department: ['研发线-效率工程部/系统研发', '新业务/海外', '审计部/税务审计'][i % 3],
    company: ['北京一笑科技发展有限公司', '北京达佳互联信息技术有限公司', '北京快手广告有限公司'][i % 3]
  });
}

ReactDOM.render(
  <Table scroll={{ x: 2000, y: 500 }} data={fixedData}>
    <Column dataKey="name" fixed="left" width={100}>
      姓名
    </Column>
    <Column dataKey="employeeNumber" fixed="left" width={120}>
      员工工号
    </Column>
    <Column dataKey="company">公司</Column>
    <Column dataKey="department">部门</Column>
    <Column dataKey="certificateNo">证件号码</Column>
    <Column dataKey="entryDate">入职日期</Column>
    <Column dataKey="leaveDate" fixed="right" width={140}>
      离职日期
    </Column>
    <Column dataKey="repayment" width={140} fixed="right" render={() => <a>查看详情</a>}>
      操作
    </Column>
  </Table>,
  mountNode
);
```
```

## 13. renderSelectionCell.md - 自定义选择框

```markdown
---
category: 基本组件
order: 12
title: 自定义选择框
---

### 自定义选择框

rowSelection 中的 columnTitle 以及 renderCell 可以自定义选择框

- columnTitle：自定义表头选择框标题
- renderCell：自定义渲染勾选框，用法与 Column 的 render 相同，会抛出以下几个参数
  checked：选中状态，
  record：当前行数据，
  index：当前行序数，
  originCell：原始选择框，

```jsx
import { useEffect, useState, useCallback } from 'react';
import { Table, Tooltip } from '@toft/react';
const { Column } = Table;

const data = [
  {
    key: '111',
    firstName: 'John',
    lastName: 'Brown',
    age: 32,
    address: 'New York No. 1 Lake Park'
  },
  {
    key: '222',
    firstName: 'Jim',
    lastName: 'Green',
    age: 42,
    address: 'London No. 1 Lake Park'
  },
  {
    key: '3333',
    firstName: 'Joe',
    lastName: 'Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park'
  }
];

function Demo() {
  const [selectedKeys, setSelectedKeys] = useState([]);

  const rowSelection = {
    selectable: true,
    renderCell: (checked, record, index, originNode) => {
      return <>{index === 0 ? <>{originNode}自定义</> : originNode}</>;
    },
    columnTitle: '自定义',
    width: 80,
    selectedKeys,
    onSelectChange: (record?: RecordType, selectedKeys?: (string | number)[]) => {
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ selectedKeys', selectedKeys);
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ record', record);
    },
    onSelect: (rowKey, selected, record) => {
      console.log(`rowKey: ${rowKey}`, 'selected: ', selected, record);
      if (selected) {
        setSelectedKeys((pre) => [...pre, rowKey]);
      } else {
        setSelectedKeys((pre) => pre.filter((item) => item !== rowKey));
      }
    },
    onSelectAll: (selected?: boolean, selectedKeys?: RecordType[], selectedRows: RecordType[]) => {
      if (selected) {
        setSelectedKeys(selectedKeys);
      } else {
        setSelectedKeys([]);
      }
    },
    getCheckboxProps: (record) => ({
      disabled: record.key === '222',
      name: record.name
    })
  };

  return (
    <Table data={data} rowSelection={rowSelection}>
      <Column dataKey="firstName">First Name</Column>
      <Column dataKey="lastName">Last Name</Column>
      <Column dataKey="age">age</Column>
      <Column dataKey="address">address</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 14. resizable.md - 可伸缩列

```markdown
---
category: 基本组件
order: 12
title: 可伸缩列
---

### 可伸缩列

配合 react-resizable 可以实现可伸缩列的效果。

```jsx
import { useState } from 'react';
import { Table } from '@toft/react';
import { Resizable } from 'react-resizable';
const { Column } = Table;

const ResizableTitle = (props) => {
  const { onResize, width, ...restProps } = props;

  if (!width) {
    return <th {...restProps} />;
  }

  return (
    <Resizable
      width={width}
      height={0}
      onResize={onResize}
      draggableOpts={{ enableUserSelectHack: false }}
    >
      <th {...restProps} />
    </Resizable>
  );
};

const originColumns = [
  {
    title: '姓名',
    dataKey: 'name',
    width: 120
  },
  {
    title: '工号',
    dataKey: 'employeeNumber',
    width: 100
  },
  {
    title: '身份证号',
    dataKey: 'certificateNo',
    width: 180
  },
  {
    title: '部门',
    dataKey: 'department'
  }
];

function Demo() {
  const realColumns = originColumns.map((column, index) => {
    if (column.width) {
      return {
        ...column,
        key: column.dataKey,
        onHeaderCell: (col) => ({
          width: col.width,
          onResize: handleResize(index)
        })
      };
    }
    return { ...column, key: column.dataKey };
  });
  const [columns, setColumns] = useState(realColumns);

  function handleResize(index) {
    console.log("🚀 ~ handleResize ~ index:", index)
    return (e, { size }) => {
      setColumns((prevColumns) => {
        const nextColumns = [...prevColumns];
        nextColumns[index] = {
          ...nextColumns[index],
          width: size.width
        };
        return nextColumns;
      });
    };
  }
  const dataSource = [
    {
      key: '1',
      name: '张三',
      employeeNumber: '43568',
      certificateNo: '230301199710225113',
      department: '研发线-效率工程部/系统研发'
    },
    {
      key: '2',
      name: '李四',
      employeeNumber: '17290',
      certificateNo: '370881199704030715',
      department: '新业务/海外'
    },
    {
      key: '3',
      name: '王五',
      employeeNumber: '17292',
      certificateNo: '513436200012188184',
      department: '审计部/税务审计'
    }
  ];

  const components = {
    header: {
      cell: ResizableTitle
    }
  };

  return (
    <Table
      data={dataSource}
      bordered
      rowClassName="k-table-row"
      components={components}
      className="table-demo-resizable-column"
    >
      {columns.map((item) => (
        <Column key={item.key} {...item}>
          {item.title}
        </Column>
      ))}
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```

```css
.table-demo-resizable-column .react-resizable {
  position: relative;
  background-clip: padding-box;
}

.table-demo-resizable-column .react-resizable-handle {
  position: absolute;
  right: -5px;
  bottom: 0;
  z-index: 1;
  width: 10px;
  height: 100%;
  cursor: col-resize;
}
```
```

## 15. selctions.md - 选择和操作

```markdown
---
category: 基本组件
order: 11
title: 选择和操作
---

### 选择和操作

往 Table 传入 rowSelection 即可打开此功能。

- 点击表头的选择框，会选择 dataSource 里所有不是 disabled 状态的行
- 选择所有行回调函数为 onSelectAll；
- 点击行的选择框会选中当前行。它的回调函数为 onSelect；
- 注意：请务必为每行数据提供一个与其他行值不同的 key，或者使用 rowKey 参数指定一个作为主键的属性名。
- rowKey 类型可以为 string，number，或函数，函数中返回当前行数据,可以通过(records)=>records.xxx 来指定 key 值

```jsx
import { useEffect, useState, useCallback } from 'react';
import { Table } from '@toft/react';
const { Column } = Table;

const data = [
  {
    key: '111',
    firstName: 'John',
    lastName: 'Brown',
    age: 32,
    address: 'New York No. 1 Lake Park'
  },
  {
    key: '222',
    firstName: 'Jim',
    lastName: 'Green',
    age: 42,
    address: 'London No. 1 Lake Park'
  },
  {
    key: '3333',
    firstName: 'Joe',
    lastName: 'Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park'
  }
];

function Demo() {
  const [selectedKeys, setSelectedKeys] = useState([]);

  const rowSelection = {
    selectable: true,
    selectedKeys,
    onSelectChange: (record?: RecordType, selectedKeys?: (string | number)[]) => {
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ selectedKeys', selectedKeys);
      console.log('🚀 ~ file: selctions.md ~ line 55 ~ Demo ~ record', record);
    },
    onSelect: (rowKey, selected, record) => {
      console.log(`rowKey: ${rowKey}`, 'selected: ', selected, record);
      if (selected) {
        setSelectedKeys((pre) => [...pre, rowKey]);
      } else {
        setSelectedKeys((pre) => pre.filter((item) => item !== rowKey));
      }
    },
    onSelectAll: (selected?: boolean, selectedKeys?: RecordType[], selectedRows: RecordType[]) => {
      if (selected) {
        setSelectedKeys(selectedKeys);
      } else {
        setSelectedKeys([]);
      }
    },
    getCheckboxProps: (record) => ({
      disabled: record.key === '222',
      name: record.name
    })
  };
  return (
    <Table data={data} rowSelection={rowSelection}>
      <Column dataKey="firstName">First Name</Column>
      <Column dataKey="lastName">Last Name</Column>
      <Column dataKey="age">age</Column>
      <Column dataKey="address">address</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 16. size.md - 不同尺寸的表格

```markdown
---
category: 基本组件
order: 5
title: 不同尺寸的表格
---

### 不同尺寸的表格

可以通过 size 来指定不同尺寸的表格

```jsx
import { useState } from 'react';
import { Table, Radio } from '@toft/react';
const { Column, ColumnGroup } = Table;

const data = [
  {
    key: '1',
    firstName: 'John',
    lastName: 'Brown',
    age: 32,
    address: 'New York No. 1 Lake Park'
  },
  {
    key: '2',
    firstName: 'Jim',
    lastName: 'Green',
    age: 42,
    address: 'London No. 1 Lake Park'
  },
  {
    key: '3',
    firstName: 'Joe',
    lastName: 'Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park'
  }
];
const Demo = () => {
  const [size, setSize] = useState('middle');
  return (
    <div>
      <Radio.Group
        value={size}
        onChange={(value) => {
          setSize(value);
        }}
      >
        <Radio.Button buttonStyle="outline" value="small">
          小尺寸表格：
        </Radio.Button>
        <Radio.Button buttonStyle="outline" value="middle">
          正常尺寸表格：
        </Radio.Button>
        <Radio.Button buttonStyle="outline" value="large">
          大尺寸表格：
        </Radio.Button>
      </Radio.Group>
      <br />
      <Table data={data} size={size}>
        <Column dataKey="firstName">First Name</Column>
        <Column dataKey="lastName">Last Name</Column>
        <Column dataKey="age">age</Column>
        <Column dataKey="address">address</Column>
      </Table>
    </div>
  );
};

ReactDOM.render(<Demo />, mountNode);
```
```

## 17. tree.md - 树状表格

```markdown
---
category: 基本组件
order: 8
title: 树状表格
---

### 树状表格

表格支持树形数据的展示，当数据中有 children 字段时会自动展示为树形表格，如果不需要或使用其他字段可以用 childrenKey 进行配置
expandedKeys 完全受控，需要传入 expandedKeys，组件内会根据传入的 expandedKeys 来展开收起，

**注意：** 请务必为每行数据提供一个与其他行值不同的 "key"，或者使用 rowKey 参数指定一个作为主键的属性名。

```jsx
import { useState } from 'react';
import { Table, Icon } from '@toft/react';
const { Column } = Table;

const data = [
  {
    key: '1',
    name: '张三',
    employeeNumber: '43568',
    certificateNo: '230301199710225113',
    department: '研发线-效率工程部/系统研发',
    child: [
      {
        key: '1-2',
        name: '张三2370881199704030715370881199704030715370881199704030715',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外',
        child: [
          {
            key: '1-2-1',
            name: '张三3370881199704030715370881199704030715',
            employeeNumber: '17290',
            certificateNo: '370881199704030715',
            department: '新业务/海外'
          }
        ]
      }
    ]
  },
  {
    key: '2',
    name: '李四',
    employeeNumber: '17290',
    certificateNo: '370881199704030715',
    department: '新业务/海外',
    child: [
      {
        key: '2-1',
        name: '李四2',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外',
        child: [
          {
            key: '1-2-1',
            name: '张三3',
            employeeNumber: '17290',
            certificateNo: '370881199704030715',
            department: '新业务/海外'
          }
        ]
      },
      {
        key: '2-2',
        name: '李四3',
        employeeNumber: '17290',
        certificateNo: '370881199704030715',
        department: '新业务/海外'
      }
    ]
  },
  {
    key: '3',
    name: '王五',
    employeeNumber: '17292',
    certificateNo: '513436200012188184',
    department: '审计部/税务审计'
  }
];
function Demo() {
  const [expandedKeys, setExpandedKeys] = useState([]);

  const onExpandChange = (expanded, record) => {
    let expandKeys = [...expandedKeys];
    if (expanded) {
      expandKeys.push(record.key);
    } else {
      expandKeys = expandKeys.filter((item) => item !== record.key);
    }
    setExpandedKeys(expandKeys);
  };

  const onExpandedRowsChange = (rows) => {
    // console.log('🚀 ~ file: tree.md ~ line 88 ~ onExpandedRowsChange ~ rows', rows);
  };

  return (
    <Table
      data={data}
      rowExpansion={{
        childrenKey: 'child',
        expandedKeys,
        onExpand: onExpandChange,
        onExpandedRowsChange,
        indentSize: 12
      }}
    >
      <Column dataKey="name" width={200}>
        姓名
      </Column>
      <Column dataKey="employeeNumber">员工工号</Column>
      <Column dataKey="department">部门</Column>
      <Column dataKey="certificateNo">证件号码</Column>
    </Table>
  );
}
ReactDOM.render(<Demo />, mountNode);
```
```

## 18. virtualTable.md - 虚拟滚动

```markdown
---
category: 基本组件
order: 0
title: 虚拟滚动
---

### 虚拟滚动

通过自定义 components 中的 body 进行虚拟滚动的实现

```jsx
import { useState, useMemo, useRef, useEffect } from 'react';
import { Table, Input } from '@toft/react';
import ResizeObserver from 'rc-resize-observer';
import { VariableSizeGrid as Grid } from 'react-window';
import classNames from 'classnames';

const { Column, ColumnGroup } = Table;

export const Data = Array(1000)
  .fill(null)
  .map((_, rowIndex) => ({
    index: rowIndex,
    children: [
      {
        index: rowIndex + '_children',
        isChildren: true,
        ...Object.fromEntries(
          Array(1000)
            .fill(null)
            .map((item, columnIndex) => [`name${columnIndex}`, `${rowIndex}:${columnIndex}`])
        )
      }
    ],
    ...Object.fromEntries(
      Array(1000)
        .fill(null)
        .map((item, columnIndex) => [`name${columnIndex}`, `${rowIndex}:${columnIndex}`])
    )
  }));

const VirtualTable = () => {
  const [data, setData] = useState(Data);
  const [expandedKeys, setExpandedKeys] = useState([]);

  const handleChange = function (rowIndex, dataKey, nextValue) {
    setData((prev) => {
      return prev.map((item, index) => {
        if (rowIndex === index) {
          return {
            ...item,
            [dataKey]: nextValue
          };
        }
        return item;
      });
    });
  };

  const columns = Array(100)
    .fill(null)
    .map((_, index) => ({
      fixed: index === 1 ? 'left' : index === 99 ? 'right' : false,
      label: index,
      width: 100 + index,
      dataKey: `name${index}`,
      align: 'right',
      colSpan: index === 2 ? 2 : index === 3 ? 0 : 1,
      onCell: (_, rowIndex) => {
        if (index !== 2) return {};
        if (rowIndex === 2) {
          return {
            rowSpan: 2
          };
        }
        if (rowIndex === 3) {
          return {
            rowSpan: 0
          };
        }
        return {};
      },
      render: (item, record, rowIndex) => {
        if (record.isChildren) {
          return item;
        }
        return (
          <Input
            value={item}
            onChange={(event) => {
              handleChange(rowIndex, `name${index}`, event.target.value);
            }}
          />
        );
      }
    }));
  const [selectedKeys, setSelectedKeys] = useState([]);

  return (
    <Table
      bordered
      data={data}
      rowKey="index"
      rowExpansion={{
        onExpandedRowsChange: setExpandedKeys,
        expandedKeys
      }}
      rowSelection={{
        selectable: true,
        selectedKeys,
        fixed: true,
        onSelectChange(_, selectedKeys) {
          setSelectedKeys(selectedKeys || []);
        }
      }}
      virtualConfig={{
        virtual: true,
        columnVirtual: true,
        rowHeight: 50,
        height: 500
      }}
    >
      {columns.map((item, index) => {
        if (index === 4) {
          return (
            <ColumnGroup key={item.dataKey + '_GROUP'} label="Group">
              <Table.Column dataKey={item.dataKey} key={item.dataKey + '_1'} {...item} />
              <Table.Column dataKey={item.dataKey} key={item.dataKey + '_2'} {...item} />
            </ColumnGroup>
          );
        }
        return <Table.Column dataKey={item.dataKey} key={item.dataKey} {...item} />;
      })}
    </Table>
  );
};

ReactDOM.render(<VirtualTable />, mountNode);
```
```

---

所有18个 .md 文件的完整内容已上述方式呈现。每个文件都包含了完整的代码示例和说明文档，涵盖了 Table 组件的各种用途：

1. **basic.md** - 基础表格使用
2. **child-table.md** - 嵌套子表格
3. **colSpanfixColumn.md** - 单元格合并并固定列
4. **colspan.md** - 表头分组和行列合并
5. **customColumn.md** - 自定义单元格渲染
6. **customEvent.md** - 自定义行/单元格事件
7. **customTable.md** - 表头吸顶效果
8. **drag.md** - 拖拽排序表格
9. **dynamicTable.md** - 动态控制表格
10. **expand.md** - 可展开表格
11. **fetchData.md** - 接口数据获取
12. **fixColumn.md** - 固定头和列
13. **renderSelectionCell.md** - 自定义选择框
14. **resizable.md** - 可伸缩列
15. **selctions.md** - 行选择操作
16. **size.md** - 表格尺寸控制
17. **tree.md** - 树状表格
18. **virtualTable.md** - 虚拟滚动
