# Phase_Recovery

从3D TIFF数据中提取以下信息：

- 小球3D位置（x, y, z坐标）

- 每个位置对应的像差Zernike系数

**输出**：
Bead_ID	X_pos	Y_pos	Z_pos	Z1 (Piston)	Z2 (Tilt_X)	Z3 (Tilt_Y)	...	Z36 (高阶像差)

## 整体流程梳理

### Step 1: 数据预处理
输入数据：50个batch的3D TIFF文件，每个batch为不同z位置的强度图像（假设为[Z1, Z2, ..., Z50]）。

**关键操作**：

- Bead定位：在每个batch中检测所有小球的2D中心位置（x, y）。

方法：使用峰值检测（如Laplacian of Gaussian）或阈值分割 + 质心计算。

- Bead匹配：跨batch关联同一小球在不同z位置的光斑（建立Bead_ID）。

方法：基于空间邻近性（如最近邻算法）或光斑特征（如强度分布）。

### Step 2: 多高度相位检索

目标：对每个Bead_ID，从其多高度光斑强度中恢复相位，进而计算Zernike系数。

**关键操作**：

- ROI提取：对每个Bead_ID，在所有batch中裁剪其光斑区域（如32×32像素）。

- 相位恢复：

算法选择：基于多高度Gerchberg-Saxton（GS）或混合输入输出（HIO）算法。

物理模型：利用标量衍射理论（如角谱法）模拟光场传播。

- 像差分解：

将恢复的相位（波前）投影到Zernike多项式基上，拟合系数（最小二乘法）。

### Step 3: 数据整合与输出
合并结果：将所有Bead_ID的位置（x, y, z）和Zernike系数整合到CSV。

- 质量控制：

剔除低信噪比（SNR）的Bead（如强度过弱或光斑不完整）。

检查Zernike系数的物理合理性（如离焦Z4与z位置的单调性）。
