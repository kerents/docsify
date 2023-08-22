# DASH应用，创建一个可供浏览器在线使用的InVest模型web应用

`Dash Plotly`是一个基于Python的开源框架，用于创建交互式、美观的Web应用程序和仪表盘。它结合了Plotly的强大数据可视化功能和Dash的Web应用开发能力，使用户能够以Python编写数据驱动的Web应用程序。
一个Dash应用通常包含以下几个部分：

- 布局（Layout）：布局定义了应用程序的外观和组件的排列方式。Dash使用HTML和CSS来创建布局，可以使用预定义的布局组件（如容器、行、列）来构建应用程序的结构。布局还可以包含Dash核心组件（如图表、表格、滑块等）以及自定义的HTML元素和样式。

- 回调函数（Callbacks）：回调函数是Dash应用程序的核心部分，用于处理用户的输入和更新应用程序中的组件。通过回调函数，可以将输入（如按钮点击、滑块拖动、下拉菜单选择等）与输出（如图表更新、数据计算、文本显示等）进行关联。回调函数通过装饰器或回调函数注册方法来定义，并使用Dash提供的回调函数装饰器来指定输入和输出的关系。

- 组件（Components）：Dash提供了一系列可用的组件，用于创建交互式的用户界面。这些组件包括核心组件（如图表、表格、滑块、下拉菜单等）和HTML组件（如文本、图像、按钮等）。组件可以通过属性设置来自定义其外观和行为，例如样式、标签、默认值等。

- 应用程序实例化（Application instantiation）：在创建Dash应用程序时，需要实例化一个Dash应用对象，并将布局和回调函数传递给该对象。这样，Dash框架就能够根据定义的布局和回调函数来构建应用程序的用户界面和交互逻辑。

- 启动应用程序（Application execution）：一旦应用程序被实例化，就可以通过运行应用程序的服务器来启动应用程序。Dash应用程序使用内置的开发服务器或其他支持WSGI协议的服务器来提供应用程序的Web界面。一旦服务器启动，应用程序就可以在浏览器中访问并与用户进行交互。

我们下面用dash创建一个InVestWeb应用

## 城市降温模型——dash应用

## 导入所需要的库

```python
import dash
import dash_uploader as du
import dash_bootstrap_components as dbc
from dash import html
from dash.dependencies import Input, Output, State
from dash import dash_table
from dash import dcc
```

```python
import warnings
warnings.filterwarnings('ignore')
import csv
import shutil 
from natcap.invest import urban_cooling_model
import os
import rioxarray as rxr
from matplotlib_scalebar.scalebar import ScaleBar
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import geopandas as gpd
import matplotlib
```

## 配置默认参数和本地文件路径

```python
myPath= 'H:\\code\\dash_test\dash_uc\\temp\\myupload' + '\\' # 默认上传文件夹
workspace_dir= 'H:\\code\\dash_test\\dash_uc\\temp\\result' # 默认工作区文件夹

# 默认参数
avg_rel_humidity = 30
cc_weight_albedo = 0.2 
cc_weight_eti = 0.2
cc_weight_shade = 0.6
green_area_cooling_distance = 1000
t_air_average_radius = 2000
t_ref = 21.5
uhi_max = 3.5
```

## 创建一个应用实例，并添加初始组件

```python
app = dash.Dash(__name__)

# 配置上传文件夹
du.configure_upload(app, folder='temp')

app.layout = html.Div(
    dbc.Container(
        [
        html.H1('城市降温模型——dash应用'),

        # 手动输入参数
        html.Ul([
        html.Li(
            dcc.Input(id='avg_rel_humidity', placeholder="平均相对湿度",value='', type='text')
            ),

        html.Li(
            dcc.Input(id='cc_weight_albedo', value='', placeholder="反照率权重",type='text')
            ),

        html.Li(
            dcc.Input(id='cc_weight_eti', value='', placeholder="反照率权重",type='text')
            ),

        html.Li(
            dcc.Input(id='cc_weight_shade', value='', placeholder="阴影权重",type='text')
            ),

        html.Li(
            dcc.Input(id='green_area_cooling_distance', value='', placeholder="植被降温半径",type='text')
            ),

        html.Li(
            dcc.Input(id='t_air_average_radius', value='', placeholder="空气混合距离",type='text')
            ),
            
        html.Li(
            dcc.Input(id='t_ref', value='', placeholder="参考空气温度",type='text')
            ),

        html.Li(
            dcc.Input(id='uhi_max', value='', placeholder="最大冷却距离",type='text')
            ),

        ]),

        html.Button('提交', id='input_button', n_clicks=0),
        html.Div(id='output-container'),



        # ————分割线————
        html.Hr(),
        du.Upload(id='aoi_vector', 
                  max_files=6,
                  text='点击或拖动文件到此上传aoi_vector',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['shp','cpg','dbf','prj','qpj','shx'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),

        html.H5('上传中或还未上传文件！', id='upload_status1'),

        # ————分割线————
        html.Hr(),
        du.Upload(id='building_vector',
                  max_files=6,
                  text='点击或拖动文件到此上传building_vector',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['shp','cpg','dbf','prj','qpj','shx'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),
        html.H5('上传中或还未上传文件！', id='upload_status3'),


        # ————分割线————
        html.Hr(),
        du.Upload(id='lulc_raster',
                  text='点击或拖动文件到此上传lulc_raster',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['tif'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),
        html.H5('上传中或还未上传文件！', id='upload_status5'),
        # ————分割线————
        html.Hr(),
        du.Upload(id='ref_eto_raster',
                  text='点击或拖动文件到此上传ref_eto_raster',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['tif'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),
        html.H5('上传中或还未上传文件！', id='upload_status6'),

        # ————分割线————
        html.Hr(),
        du.Upload(id='biophysical_table',
                  text='点击或拖动文件到此上传biophysical_table',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['csv'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),
        html.H5('上传中或还未上传文件！', id='upload_status2'),
        html.Div(id='biophysical_table_graph'),

        # ————分割线————
        html.Hr(),
        du.Upload(id='energy_consumption_table',
                  text='点击或拖动文件到此上传energy_consumption_table',
                  text_completed='已完成上传文件：',
                  cancel_button=True,
                  pause_button=True,
                  filetypes=['csv'],
                  default_style={
                  'background-color': '#fafafa',
                  'font-weight': 'bold'
                  },
                  upload_id= myPath
                ),
        html.H5('上传中或还未上传文件！', id='upload_status4'),
        html.Div(id='energy_consumption_table_graph'),
        

        html.Button('运行计算', id='submit_button', style= {
        'width': '100%',
        'background-color': '#d888dd',
        'color': 'white',
        'padding': '14px 20px',
        'margin': '8px 0',
        'border': 'none',
        'border-radius': '4px',
        'cursor': 'pointer',
      },
      ),
        html.Div(id='output1'),
        html.Div(id='output2'),
        
        
        ]
    ),style={'padding':'25px 300px'}
)
```

## 执行上传配置参数的回调函数

```python
@app.callback(
    Output('output-container', 'children'),
    [Input('input_button', 'n_clicks')],
    [
        State('avg_rel_humidity', 'value'),
        State('cc_weight_albedo', 'value'),
        State('cc_weight_eti', 'value'),
        State('cc_weight_shade', 'value'),
        State('green_area_cooling_distance', 'value'),
        State('t_air_average_radius', 'value'),
        State('t_ref', 'value'),
        State('uhi_max', 'value'),
    ]
)
def update_value(n_clicks, *input_values):
    if n_clicks > 0:
        # 将变量转换为全局变量
        global avg_rel_humidity, cc_weight_albedo, cc_weight_eti, cc_weight_shade, green_area_cooling_distance, t_air_average_radius , t_ref, uhi_max
        avg_rel_humidity, cc_weight_albedo, cc_weight_eti, cc_weight_shade, green_area_cooling_distance, t_air_average_radius , t_ref, uhi_max = input_values
        return html.H3('参数配置成功')
```

## 执行上传操作的回调函数

可以创建一个通用的回调函数，然后在循环中为每个需要的输入和输出组合注册这个回调。这样的话，只需要写一次回调函数，然后把需要的输入和输出作为循环变量传入即可。  

在下面的代码中，我们首先定义了一个`register_callback`函数，它接受一个app实例和回调的输入/输出ID。  

然后，创建一个回调，并使用传入的ID作为输入/输出。然后，创建一个列表`ids`来保存所有需要注册回调的ID。  

最后，我们使用一个for循环来为列表中的每个ID注册回调。这样就可以用一种更加简洁和优雅的方式来注册所有的回调。  

如果需要添加或删除回调，只需要修改`ids`列表

```python
# 定义一个注册回调函数的方法
def register_callback(app, output_id, input_id):
    # 装饰器定义一个回调函数，接受一个输入和一个状态
    @app.callback(
        # 回调函数的输出为指定的组件的'children'属性
        Output(output_id, 'children'),
        # 回调函数的输入为指定的组件的'isCompleted'属性
        Input(input_id, 'isCompleted'),
        # 回调函数的状态为指定的组件的'fileNames'属性
        State(input_id, 'fileNames'),
    )
    # 回调函数的定义，接受两个参数
    def show_upload_status(isCompleted, fileNames):
        # 如果上传成功，则返回文件的路径
        if isCompleted:
            # 指定文件的路径为固定格式下的指定目录和上传文件的文件名
            path= myPath + fileNames[0]
            print('上传了文件'+ fileNames[0])
            return '已完成上传，文件路径：'+path
        # 如果上传还未完成，则返回上传状态
        return '上传中或还未上传文件！'

# 待注册的回调 ID 列表
ids = ['aoi_vector', 'biophysical_table', 'building_vector', 'energy_consumption_table', 'lulc_raster', 'ref_eto_raster']

# 循环注册回调函数，返回上传状态
for i, id in enumerate(ids):
    # 注册回调函数，传入应用程序、回调函数的输出和输入的ID
    register_callback(app, f'upload_status{i+1}', id)
```

## 显示表格的回调函数

```python
# 编辑csv文件回调函数
@app.callback(    
    Output('biophysical_table_graph', 'children',),
    Input('biophysical_table', 'isCompleted'),
    State('biophysical_table', 'fileNames'),
    )

def biophysical_table_graph(isCompleted, fileNames):
        if isCompleted:
            # 指定文件的路径为固定格式下的指定目录和上传文件的文件名
            path=myPath + fileNames[0]
            df = pd.read_csv(path)
            table = html.H3('生物物理表'),dash_table.DataTable(
                id='Bio_table',
                columns=[{"name": i, "id": i} for i in df.columns],
                data=df.to_dict('records'),
                editable=True,),html.Button('保存',id='biophysical_save'),html.Div(id='save-status1')
            return table

# 编辑csv文件回调函数
@app.callback(    
    Output('energy_consumption_table_graph', 'children',),
    Input('energy_consumption_table', 'isCompleted'),
    State('energy_consumption_table', 'fileNames'),
    )

def energy_consumption_table_graph(isCompleted, fileNames):
        if isCompleted:
            # 指定文件的路径为固定格式下的指定目录和上传文件的文件名
            path=myPath + fileNames[0]
            df = pd.read_csv(path)
            table =html.H3('建筑能源消耗表'), dash_table.DataTable(
                id='Eng_table',
                columns=[{"name": i, "id": i} for i in df.columns],
                data=df.to_dict('records'),
                editable=True,),html.Button('保存',id='energy_save'),html.Div(id='save-status2')
            return table
```

## 修改表格内容的回调函数

```python
@app.callback(
    Output('save-status1', 'children'),
    Input('biophysical_save', 'n_clicks'),
    State('Bio_table', 'data')
)
def save_data(n_clicks, data):
    if n_clicks is None:
        return html.H3('点击按钮保存文件')
    else:
        # print(type(data))
        # print(data)
        keys = data[0].keys() 
        filename =myPath + 'Biophysical_UHI_fake.csv'  # 文件路径 + CSV文件名

        with open(filename, 'w', newline='') as csvfile:
            writer = csv.DictWriter(csvfile, fieldnames=keys)
            writer.writeheader()  # 写入CSV文件的标题行
            writer.writerows(data)  # 写入数据行
        return html.H3('已保存')
    
@app.callback(
    Output('save-status2', 'children'),
    Input('energy_save', 'n_clicks'),
    State('Eng_table', 'data')
)
def save_data(n_clicks, data):
    if n_clicks is None:
        return html.H3('点击按钮保存文件')
    else:
        # print(type(data))
        # print(data)
        keys = data[0].keys() 
        filename =myPath + 'Fake_energy_savings.csv'  # 文件路径 + CSV文件名

        with open(filename, 'w', newline='') as csvfile:
            writer = csv.DictWriter(csvfile, fieldnames=keys)
            writer.writeheader()  # 写入CSV文件的标题行
            writer.writerows(data)  # 写入数据行
        return html.H3('已保存')
```

## 执行InVest的回调函数

```python
# 显示正在运行
@app.callback(    
    Output('output1', 'children',),
    Input('submit_button', 'n_clicks'),)

def run_usda_esv(n_clicks):
    if n_clicks:
        return html.H1('正在运行中……')
    return html.Div()

# ————————————————————————————————
@app.callback(
    Output('output2', 'children'),
    Input('submit_button', 'n_clicks'),
    State('aoi_vector', 'fileNames'),
    State('biophysical_table', 'fileNames'),
    State('building_vector', 'fileNames'),
    State('energy_consumption_table', 'fileNames'),
    State('lulc_raster', 'fileNames'),
    State('ref_eto_raster', 'fileNames'),
)

def run_usda_esv(n_clicks, aoi_vector, biophysical_table, building_vector, energy_consumption_table, lulc_raster, ref_eto_raster):
    if n_clicks:
        # 获取所有文件的路径
        args = {  # 字典里的路径需要使用绝对路径
            "aoi_vector_path": myPath + aoi_vector[0],
            "biophysical_table_path": myPath + biophysical_table[0],
            "building_vector_path": myPath + building_vector[0],
            "energy_consumption_table_path": myPath + energy_consumption_table[0],
            "lulc_raster_path": myPath + lulc_raster[0],
            "ref_eto_raster_path": myPath + ref_eto_raster[0],
            "avg_rel_humidity": avg_rel_humidity,
            "cc_method": "factors",
            "cc_weight_albedo": cc_weight_albedo,
            "cc_weight_eti": cc_weight_eti,
            "cc_weight_shade": cc_weight_shade,
            "do_energy_valuation": True,
            "do_productivity_valuation": True,
            "green_area_cooling_distance": green_area_cooling_distance,

            "t_air_average_radius": t_air_average_radius,
            "t_ref": t_ref,
            "uhi_max": uhi_max,
            "workspace_dir": workspace_dir
        }

        # 查看lulc、建筑轮廓和边界
        lulc=rxr.open_rasterio(args['lulc_raster_path'],masked=True).squeeze()
        building_footprints=gpd.read_file(args['building_vector_path']).to_crs(lulc.rio.crs)
        aoi=gpd.read_file(args['aoi_vector_path']).to_crs(lulc.rio.crs)
        print(f'lulc epsg:{lulc.rio.crs}\nbuilding footprints epsg:{building_footprints.crs.to_epsg()}\naoi epsg:{aoi.crs.to_epsg()}')
        f, ax=plt.subplots(1,1,figsize=(10,10))
        np.random.seed(10)
        cmap=matplotlib.colors.ListedColormap (np.random.rand ( 256,3))
        lulc.plot.imshow(ax=ax,add_colorbar=False,cmap=cmap)
        building_footprints.plot(color='none',edgecolor='k',linewidth=1,ax=ax,linestyle='-')
        aoi.plot(color='none',edgecolor='r',linewidth=2,ax=ax,linestyle='--')

        ax.add_artist(ScaleBar(1))  
        ax.set_axis_off()
        plt.tight_layout()
        plt.savefig('assets/check_out')

        # 运行 usda_esv.Urban_cooling 函数
        print ('正在运行')
        urban_cooling_model.execute(args)

        HM=rxr.open_rasterio(os.path.join(args['workspace_dir'],'hm.tif'),masked=True).squeeze()

        f,ax=plt.subplots(1,1,figsize=(10,5))
        cmap='plasma'
        HM.plot.imshow(ax=ax,add_colorbar=True,cmap=cmap)
        ax.set_title('HM')
        plt.savefig('assets/result')

        # img=Image.open('.\\check_out.png')
        # img.show()
        

        
        return html.H1('运算完成'),html.Div([html.H3('lulc、建筑轮廓和边界'),html.Img(src='assets/check_out.png', width=1000),
                                            html.H3('结果查看'),html.Img(src='assets/result.png', width=1000),
                                            html.Button("下载结果", id="btn-download",style= {
                                            'width': '100%',
                                            'background-color': '#cdcdcd',
                                            'color': 'white',
                                            'padding': '14px 20px',
                                            'margin': '8px 0',
                                            'border': 'none',
                                            'border-radius': '4px',
                                            'cursor': 'pointer',
                                            }),
                                            dcc.Download(id="download-text")]),
```

## 执行下载功能的回调函数

将结果文件压缩打包，并使用浏览器下载。

```python
@app.callback(
    Output("download-text", "data"),
    Input("btn-download", "n_clicks"),
    prevent_initial_call=True,
)
def func(n_clicks):
    shutil.make_archive('download/result','zip','./temp/result/') # 压缩结果
    return dcc.send_file(
        "download/result.zip"
    )
```

## 启动应用程序

启动应用，并配置应用开启在本地8051端口，可以自行配置SSL证书和密钥以使用https加密传输。随后可使用设备的ip地址加端口号进行访问。

```python
if __name__ == '__main__':
    app.run_server(debug=False, ssl_context=('../证书.crt', '../私钥.key'),suppress_callback_exceptions=True,port=8051,host='0.0.0.0')
```
