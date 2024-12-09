03组代码复现

jupyter实例运行在老师服务器8889端口上（防止与别组冲突） 通过 172.18.232.211:8889 访问

根目录位于

源码仓库地址：https://github.com/Chang-su0423/large_data_work_group03.git，      更改可提pr

项目依赖已导出为requirements.txt,           
pip install -r requirements.txt      可批量安装依赖





### 1. **数据加载与探索**

- **加载数据**：代码首先通过 `pd.read_csv` 加载训练集 (`train.csv`) 和测试集 (`test.csv`) 数据，并存储在 pandas DataFrame 中。
- **合并数据**：将训练集和测试集合并成一个数据集 `datas`，以便进行统一的预处理。

### 2. **数据类型识别**

- **识别类别特征**：通过 `select_dtypes` 方法识别所有类别特征（即对象类型的列）并存储在 `cat_columns` 中。
- **识别数值特征**：通过 `select_dtypes` 方法识别所有数值特征（即非对象类型的列）并存储在 `numerical_columns` 中。

### 3. **数据预处理**

- 时间特征处理

  ：

  - 将 `policy_bind_date` 和 `incident_date` 转换为日期类型，并提取出年、月、日、星期等信息，作为新特征（例如：`policy_bind_date_year`, `incident_date_month` 等）。
  - 计算日期之间的差异（例如：`policy_bind_date_diff`，表示从最早的 `policy_bind_date` 到当前日期的天数差异）。

- 交叉特征工程

  ：

  - 构造与保险类型相关的特征（如：`injury_claim_pct`, `property_claim_pct`），并根据不同的声明类型（如伤害、财产、车辆）创建交叉特征（如：`incident_type_&_is_injury_claim`）。
  - 根据事故位置、车辆信息等进一步构造特征（例如：`incident_location_type`, `incident_location_street` 等）。

- **删除不必要的列**：移除不再需要的列（如 `policy_bind_date`, `incident_date`, `incident_location`, `policy_number`, `auto_make`, `auto_model` 等）。

### 4. **特征转换**

- **类别特征标签编码**：使用 `LabelEncoder` 对所有类别特征进行标签编码，使其适应模型输入要求。
- **数值特征量化转换**：使用 `QuantileTransformer` 对数值特征进行转换，确保数据呈正态分布，提高模型表现。

### 5. **数据集划分**

- **划分训练集和测试集**：根据 `fraud_reported` 标签的缺失与否来划分训练集和测试集。
- **划分验证集**：使用 `train_test_split` 将训练集进一步划分为训练集和验证集，以便在训练过程中进行交叉验证。

### 6. **LightGBM模型训练**

- **单次训练**：使用 `LGBMClassifier` 初始化 LightGBM 模型，并在训练集上进行训练。使用早停技术（`early_stopping_rounds`）来防止过拟合。
- **模型预测**：通过 `predict_proba` 获取模型在测试集上的预测概率，并进行输出。

### 7. **交叉验证**

- **K折交叉验证**：使用 `StratifiedKFold` 进行 K 折交叉验证，将数据分成 `NFOLD` 个子集，依次进行训练和验证。
- **训练与验证**：每次折叠训练模型，并计算验证集的 AUC 值，最后将所有折叠的预测结果进行平均。

### 8. **模型评估与结果保存**

- **模型评估**：通过计算 AUC 等评估指标来评估模型性能。
- **保存结果**：将最终的预测结果保存为 CSV 文件，并提交到比赛平台。

### 总结

此代码涉及到以下几个主要任务：

1. 数据的预处理和特征工程（包括时间特征、交叉特征的构造）。
2. 使用 LightGBM 进行训练，并在训练过程中应用早停和交叉验证来避免过拟合。
3. 使用 K 折交叉验证来评估模型性能。
4. 最终将预测结果保存并提交。

通过这些步骤，可以有效地提高模型的准确性和稳定性。
