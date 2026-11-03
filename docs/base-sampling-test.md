# Base sampling test (isolated Kokon fork)

This branch changes only the KockBurn/kokon test fork. It does not modify Workbench, Control, or their server files.

1. Build/install this Kokon test branch separately from the existing working installation (do not overwrite your current installed Kokon).
2. In **Settings > Models**, select **Use my own files**, point to your Qwen Image 2.1 Base INT8 ConvRot diffusion weights, VAE, Qwen3-VL text encoder and projector.
3. Select **Sampling profile: Base**, then **Save**.
4. Under the generation screen's **Advanced** panel, set **Steps 40**, **CFG 2.0**, **Sampler Euler**. Keep the same image, prompt, seed, effective working resolution and upscaling setting as the comparison run. Disable Prompt+ and LoRAs for the initial controlled test.
5. Run one image edit. In **Settings > Diagnostics**, inspect the service log for **SAMPLING_SUBMIT**. It must report `mode=Base sample_steps=40 txt_cfg=2 custom_sigmas=0`.
6. Verify the live inference progress reaches **40/40** sampling steps. Saved output metadata should also list steps 40 and CFG 2.0. The submit log and metadata prove Kokon sent the values; only backend-side logs or an A/B image result can independently establish guidance effects.
7. For a CFG check, re-run with the **same seed**, identical inputs and Steps 40, but CFG 1.0. Confirm `txt_cfg=1` is sent. Compare results.
8. Switch back to **Turbo** on a Turbo-weight profile. Even if old advanced overrides remain in a project, effective requests must still report `sample_steps=8 txt_cfg=1 custom_sigmas=9` (exact Turbo schedule).

## Notes

- **Base mode is explicit**. A filename is not reliable identification of Base vs Turbo, and merely selecting your own weights does not automatically select Base.
- Base uses the backend's normal sampling schedule; Kokon's built-in Turbo sigmas are omitted.
- Rust tests assert that Base 40/CFG 2 is serialized into the actual `/sdcpp/v1/img_gen` payload and that Turbo is unchanged.
- Before a production install, test this fork on actual Base weights. Kokon's CUDA engine is reused, not modified.

## Automated checks

The fork's pull-request workflow runs UI typechecking/Vitest and Rust core tests. These verify serialized sampling requests and Turbo regression; they **do not** verify RTX 3090 image quality or backend-side application of CFG. Check the workflow result before installing the branch. The installed production Kokon and other repositories stay unchanged.
