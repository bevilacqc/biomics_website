## Context
Brillouin microscopy is rapidly emerging as a powerful technique for imaging the mechanical properties of biological specimens in a label-free, non-contact manner. Different approaches can be used to acquire the Brillouin signal (e.g. spontaneous, stimulated, time-domain) but ultimately they all collect a spectrum for each voxel in the image. For more information you can find many reviews about the topic, e.g. [Kabakova et al 2024](https://doi.org/10.1038/s43586-023-00286-z) and [Prevedel et al. 2019](https://doi.org/10.1038/s41592-019-0543-3).

Despite its growing adoption, standardized methods for storing, sharing, and analyzing spectral data and associated metadata are still lacking. We recently proposed a standardized file format based on ZARR, together with a web app, [BrimView](https://biobrillouin.org/brimview), to visualize and analyze the data. You can read more about it in [our preprint](https://doi.org/10.48550/arXiv.2509.07566).

## Aims
There are several aspects of the web app that need improvements. During the hackathon, we focused mainly on two tasks:
### Implementing data processing with GPGPU
Typical Brillouin datasets consist of >10,000 spectra. Most commonly, the data is analyzed by fitting a known function (e.g. Lorentzian) to the Brillouin peaks. This can easily take tens of minutes when using standard fitting algorithms on the CPU. We therefore wanted to explore the possibility of using GPGPU to accelerate these fits in the browser. There is one project we know that uses CUDA kernels on NVIDIA GPUs that makes it quite fast called [GPUfit](https://github.com/gpufit/Gpufit), so we considered integrating GPUfit into the web app to enable GPU-accelerated fitting in the browser.
### Extending the support of local ZARR files to ZARR directories in addition to .zip files
A [ZARR file](https://zarr.dev/) consists of an entire directory, but the current version of BrimView only supports reading zipped folders. We therefore wanted to implement support for reading directories as well.

## Achievements
### GPGPU acceleration
First, we explored how to run CUDA kernels from a website with a client's NVIDIA GPU. Our search quickly found it was impossible. Then, we resorted to using [WebGPU](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API) to implement fitting on the GPU in the browser. With the help of a Claude Code agent, we ported Gpufit to WebGPU. The results of this port are available in the [gpufit-wgpu](https://github.com/vrbouza/gpufit-wgpu) GitHub repository.
An initial integration into the BrimView app is available in a [forked repository](https://github.com/vrbouza/Brimview). Preliminary tests showed a substantial speedup of fitting (about 1 s for more than 10,000 spectra).
### Reading local directories
After exploring different options, including the [Filesystem Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) (which has limited support in major browsers) and the [MEMFS filesystem](https://emscripten.org/docs/api_reference/Filesystem-API.html#memfs) in Pyodide, we resorted to using the [webkitdirectory property](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/webkitdirectory) of `HTMLInputElement`, which allows loading a directory as a list of files that can then be mapped to a Zarr store.

## Next steps
We plan to test the new code and fully integrate it into the [existing codebase](https://github.com/prevedel-lab/BrimView).
