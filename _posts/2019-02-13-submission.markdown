---
layout: post
title:  "提交格式说明"
date:   2019-02-01 11:23:51 +0800
---

## 提交格式

选手提交HDF5格式的文件，在文件中写入一个与数据集中GroundTruth表结构相似的表，只是加入Weight一列数据，并命名为为Answer

<div markdown="0" align="center">
<table cellspacing="0" border="0">
   <colgroup width="180"></colgroup>
   <colgroup width="200"></colgroup>
   <colgroup width="180"></colgroup>
   <tr>
      <td style="border-bottom: 2px solid #000000" colspan="4" height="19" align="center" valign="middle"><b><i><font color="#000000">Answer</font></i></b></td>
   </tr>
   <tr>
      <td height="18" align="center" valign="middle"><b><font color="#000000">EventID</font></b> (int64)</td>
      <td align="center" valign="middle"><b><font color="#000000">ChannelID</font></b> (int16)</td>
      <td align="center" valign="middle"><b><font color="#000000">PETime</font></b> (int16)</td>
      <td align="center" valign="middle"><b><font color="#000000">Weight</font></b> (float32)</td>
   </tr>
   <tr>
      <td height="18" align="center" valign="middle" sdval="1" sdnum="1033;"><font color="#000000">1</font></td>
      <td align="center" valign="middle" sdval="0" sdnum="1033;"><font color="#000000">0</font></td>
      <td align="center" valign="middle"><font color="#000000">269</font></td>
      <td align="center" valign="middle"><font color="#000000">0.3</font></td>
   </tr>
   <tr>
      <td height="18" align="center" valign="middle" sdval="1" sdnum="1033;"><font color="#000000">1</font></td>
      <td align="center" valign="middle" sdval="1" sdnum="1033;"><font color="#000000">0</font></td>
      <td align="center" valign="middle"><font color="#000000">284</font></td>
      <td align="center" valign="middle"><font color="#000000">0.5</font></td>
   </tr>
   <tr>
      <td height="18" align="center" valign="middle" sdval="1" sdnum="1033;"><font color="#000000">1</font></td>
      <td align="center" valign="middle" sdval="2" sdnum="1033;"><font color="#000000">0</font></td>
      <td align="center" valign="middle"><font color="#000000">287</font></td>
      <td align="center" valign="middle"><font color="#000000">2.0</font></td>
   </tr>
   <tr>
      <td height="18" align="center" valign="middle"><font color="#000000">…</font></td>
      <td align="center" valign="middle"><font color="#000000">…</font></td>
      <td align="center" valign="middle"><font color="#000000">…</font></td>
      <td align="center" valign="middle"><font color="#000000">…</font></td>
   </tr>
</table>
</div>

## 竞赛培训文件

竞赛培训文件可在赛事平台的 "Knowledge Base" 处获得。

主站：https://data-contest.net9.org/articles/8fda5c60-8c8a-48ea-9302-5c36ebcad589
亚洲区：https://nu.airelinux.org/articles/8fda5c60-8c8a-48ea-9302-5c36ebcad589
欧洲区：https://dc.airelinux.org/articles/8fda5c60-8c8a-48ea-9302-5c36ebcad589

## 写入文件的示例

在下面的示例中，我们向文件中写入上表中的三条示例数据。

### Python
```python
# An example of writing a file.
import tables

# Define the database columns
class AnswerData(tables.IsDescription):
    EventID    = tables.Int64Col(pos=0)
    ChannelID  = tables.Int16Col(pos=1)
    PETime     = tables.Int16Col(pos=2)
    Weight     = tables.Float32Col(pos=3)

# Create the output file and the group
h5file = tables.open_file("MyAnswer.h5", mode="w", title="OneTonDetector")

# Create tables
AnswerTable = h5file.create_table("/", "Answer", AnswerData, "Answer")
answer = AnswerTable.row

# Write data 
answer['EventID'] =  1
answer['ChannelID'] = 0
answer['PETime'] = 269 
answer['Weight'] = 0.3 
answer.append()
answer['EventID'] =  1
answer['ChannelID'] = 0
answer['PETime'] = 284 
answer['Weight'] = 0.5 
answer.append()
answer['EventID'] =  1
answer['ChannelID'] = 0
answer['PETime'] = 287 
answer['Weight'] = 2.0 
answer.append()

# Flush into the output file
AnswerTable.flush()

h5file.close()
```

### C++
```
g++ -std=c++11 -o submit -I/usr/local/hdf5/include/ Submit.cpp -lhdf5 -lhdf5_hl
```

```cpp
#include "hdf5.h"
#include "hdf5_hl.h"
#include <cstdlib>
#include <iostream>
using namespace std;

constexpr size_t nFields = 4;
constexpr size_t nRecord = 3;

struct AnswerData
{
    long long EventID;
    short ChannelID;
    short PETime;
    float Weight;
};

int main() {

    AnswerData dst_buf;

    // Calculate the size and the offsets of our struct members in memory
    size_t dst_size =  sizeof( AnswerData );
    size_t dst_offset[nFields] = { HOFFSET( AnswerData, EventID ),
        HOFFSET( AnswerData, ChannelID ),
        HOFFSET( AnswerData, PETime ),
        HOFFSET( AnswerData, Weight )
    };

    size_t dst_sizes[nFields] = { sizeof( dst_buf.EventID),
        sizeof( dst_buf.ChannelID),
        sizeof( dst_buf.PETime),
        sizeof( dst_buf.Weight)
    };

    // Define an array of data
    AnswerData p_data[nRecord] = {
        {1,0, 269, 0.3},
        {1,0, 284, 0.5},
        {1,0, 287, 2.0}
    };

    // Define field information
    const char *field_names[nFields]  =
    { "EventID","ChannelID", "PETime", "Weight"};
    hid_t      field_type[nFields];
    hid_t      file_id;
    hsize_t    chunk_size = 10;
    int        *fill_data = NULL;
    int        compress  = 0;
    int        i;

    // Initialize field_type
    field_type[0] = H5T_NATIVE_LLONG;
    field_type[1] = H5T_NATIVE_SHORT;
    field_type[2] = H5T_NATIVE_SHORT;
    field_type[3] = H5T_NATIVE_FLOAT;

    // Create a new file using default properties.
    file_id = H5Fcreate( "MyAnswer_cpp.h5", H5F_ACC_TRUNC, H5P_DEFAULT, H5P_DEFAULT );


    // Make the table.
    H5TBmake_table( "Table Title", file_id, "Answer", nFields, nRecord,
                dst_size,field_names, dst_offset, field_type,
                chunk_size, fill_data, compress, p_data  );

    // Close the file.
    H5Fclose( file_id );

    return 0;
}
```
