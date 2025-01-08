diffusers==0.11.1

将github中diffusion_policy.py 去替换下载下来的robomimic-diffusion-policy-mg\robomimic-diffusion-policy-mg\robomimic\algo 中的diffusion_policy.py即可对原始diffusion-policy进行训练推理。

若可成功进行仿真，可以把diffusion-policy中567-581行注释打开，尝试一下在FiLM的基础上加上DiT的效果如何
