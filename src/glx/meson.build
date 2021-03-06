# Copyright © 2017 Intel Corporation

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

# TODO: 
#subdir('windows')

files_libglx = files(
  'clientattrib.c',
  'clientinfo.c',
  'compsize.c',
  'create_context.c',
  'eval.c',
  'glxclient.h',
  'glxcmds.c',
  'glxconfig.c',
  'glxconfig.h',
  'glxcurrent.c',
  'glx_error.c',
  'glx_error.h',
  'glxext.c',
  'glxextensions.c',
  'glxextensions.h',
  'glxhash.c',
  'glxhash.h',
  'glx_pbuffer.c',
  'glx_query.c',
  'indirect_glx.c',
  'indirect_init.h',
  'indirect_texture_compression.c',
  'indirect_transpose_matrix.c',
  'indirect_vertex_array.c',
  'indirect_vertex_array.h',
  'indirect_vertex_array_priv.h',
  'indirect_vertex_program.c',
  'indirect_window_pos.c',
  'packrender.h',
  'packsingle.h',
  'pixel.c',
  'pixelstore.c',
  'query_renderer.c',
  'render2.c',
  'renderpix.c',
  'single2.c',
  'singlepix.c',
  'vertarr.c',
)

extra_libs_libglx = []

if with_dri
  files_libglx += files(
    'dri_common.c',
    'dri_common.h',
    'dri_common_query_renderer.c',
    'dri_common_interop.c',
    'xfont.c',
    'drisw_glx.c',
    'drisw_priv.h',
  )
endif

# dri2
if with_dri and with_dri_platform == 'drm' and dep_libdrm.found()
  files_libglx += files(
    'dri2.c',
    'dri2_glx.c',
    'dri2.h',
    'dri2_priv.h',
    'dri_glx.c',
    'dri_sarea.h',
    'XF86dri.c',
    'xf86dri.h',
    'xf86dristr.h',
  )
endif

if with_dri3
  files_libglx += files('dri3_glx.c', 'dri3_priv.h')
endif

if with_appledri
  files_libglx += files('applegl_glx.c')
elif with_windowsdri
  files_libglx += files('driwindows_glx.c')
  # TODO
  #extra_libs_libglx += [
    #libwindowsdri,
    #libwindowsglx,
  #]
endif

dri_driver_dir = join_paths(get_option('prefix'), dri_drivers_path)
if not with_glvnd
  gl_lib_name = 'GL'
  gl_lib_version = '1.2.0'
else
  gl_lib_name = 'GLX_mesa'
  gl_lib_version = '0'
  files_libglx += files(
    'g_glxglvnddispatchfuncs.c',
    'g_glxglvnddispatchindices.h',
    'glxglvnd.c',
    'glxglvnd.h',
    'glxglvnddispatchfuncs.h',
  )
endif

gl_lib_cargs = [
  '-D_REENTRANT', '-DDEFAULT_DRIVER_DIR="@0@"'.format(dri_driver_dir),
]

if dep_xxf86vm != [] and dep_xxf86vm.found()
  gl_lib_cargs += '-DHAVE_XF86VIDMODE'
endif

libglx = static_library(
  'glx',
  [files_libglx, glx_generated],
  include_directories : [
    inc_common, inc_glapi, inc_loader,
    include_directories('../../include/GL/internal'),
  ],
  c_args : [c_vis_args, gl_lib_cargs,
            '-DGL_LIB_NAME="lib@0@.so.@1@"'.format(gl_lib_name, gl_lib_version.split('.')[0])],
  link_with : [libloader, libloader_dri3_helper, libmesa_util, libxmlconfig],
  dependencies : [dep_libdrm, dep_dri2proto, dep_glproto, dep_x11, dep_glvnd],
  build_by_default : false,
)

# workaround for bug #2180
dummy_c = custom_target(
  'dummy_c',
  output : 'dummy.c',
  command : [prog_touch, '@OUTPUT@'],
)

if with_glx == 'dri'
  libgl = shared_library(
    gl_lib_name,
    dummy_c,  # workaround for bug #2180
    include_directories : [
      inc_common, inc_glapi, inc_loader,
      include_directories('../../include/GL/internal'),
    ],
    link_with : [libglapi_static, libglapi],
    link_whole : libglx,
    link_args : [ld_args_bsymbolic, ld_args_gc_sections],
    dependencies : [dep_libdrm, dep_dl, dep_m, dep_thread, dep_x11,
                    dep_xcb_dri2, dep_xcb_dri3],
    version : gl_lib_version,
    install : true,
  )

  pkg.generate(
    name : 'gl',
    filebase : 'gl',
    description : 'Mesa OpenGL Library',
    version : meson.project_version(),
    libraries : libgl,
    libraries_private : gl_priv_libs,
    requires_private : gl_priv_reqs,
    variables : ['glx_tls=yes'],
  )
endif

if with_tests
  subdir('tests')
endif
