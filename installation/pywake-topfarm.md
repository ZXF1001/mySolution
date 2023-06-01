[< 返回主页](../README.md)
# PyWake和TOPFARM安装
---
## 安装PyWake
1. 创建环境
    ```bash
    conda create -n pywake python=3.8
    conda activate pywake
    ```
2. （[官网安装说明](https://topfarm.pages.windenergy.dtu.dk/PyWake/installation.html)）直接pip安装
    ```bash
    pip install py_wake
    ```
3. 验证安装
如果想验证一下是否安装成功，可以执行以下步骤：

   1. 进入python环境
        ```bash
        python
        ```
   2. 在python命令行测试是否能成功运行quickstart案例
        ```python
        # 测试整个包能否import成功，耗时可能较长
        import py_wake

        # 进行一个quickstart算例测试
        from py_wake.examples.data.hornsrev1 import Hornsrev1Site, V80, wt16_x, wt16_y
        from py_wake import NOJ
        windTurbines = V80()
        site = Hornsrev1Site()
        noj = NOJ(site,windTurbines)
        simulationResult = noj(wt16_x,wt16_y)
        simulationResult.aep()
        # 成功输出发电量数据数据，表示安装成功
        ```

## （可选，需要使用优化功能时安装）安装TOPFARM
1. 切换环境
    ```bash
    conda activate pywake
    ```
2. 安装（[官网安装说明](https://topfarm.pages.windenergy.dtu.dk/TopFarm2/installation.html)）
    ```bash
    pip install topfarm
    ```
3. 验证安装
  
    如果想验证一下是否安装成功，可以执行以下步骤：

   1. 进入python环境
      ```bash
      python
      ```
   2. 在python命令行测试是否能成功运行quickstart案例
      ```python
      # 测试整个包能否import成功，耗时可能较长
      import topfarm
      
      import numpy as np
      from py_wake.deficit_models.gaussian import IEA37SimpleBastankhahGaussian
      from py_wake.examples.data.iea37 import IEA37_WindTurbines, IEA37Site
      from topfarm.cost_models.py_wake_wrapper import PyWakeAEPCostModelComponent
      from topfarm import TopFarmProblem
      from topfarm.easy_drivers import EasyScipyOptimizeDriver
      from topfarm.examples.iea37 import get_iea37_initial, get_iea37_constraints, get_iea37_cost
      from topfarm.plotting import NoPlot
      n_wt = 9
      n_wd = 16
      site = IEA37Site(9)
      wind_turbines = IEA37_WindTurbines()
      wd = np.linspace(0.,360.,n_wd, endpoint=False)
      wfmodel = IEA37SimpleBastankhahGaussian(site, wind_turbines)
      cost_comp = PyWakeAEPCostModelComponent(wfmodel, n_wt, wd=wd)
      initial = get_iea37_initial(n_wt)
      driver = EasyScipyOptimizeDriver()
      design_vars = dict(zip('xy', (initial[:, :2]).T))
      tf_problem = TopFarmProblem(
                  design_vars,
                  cost_comp,
                  constraints=get_iea37_constraints(n_wt),
                  driver=driver,
                  plot_comp=NoPlot())
      _, state, _ = tf_problem.optimize()
      # 成功输出优化结果，表示安装成功
      ```
    3. 测试完成后，会在当前路径下产生一个reports文件夹和一个openmdao文件，可以删掉 
