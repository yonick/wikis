
#Secure Data Path

##Overview

SDP (Secure Data Path) is the mechanism that can securely transfer private data between modules, SDP will be the key point that make DRM solution actually secure, because we don't want the protected media data exposing to any user, all the operations related to the private data (media data & user certificates) processing should be protected under a secure environment (TPM?TEE?), users cannot directly access to them. Let's take a look with the mail thread: https://groups.google.com/forum/#!topic/opencdm/ovamIrJn3z0

**SFO15-205 - OP-TEE Content Decryption with Microsoft PlayReady on ARM TrustZone.pdf**

Apparently, the implementation of SDP will have a impact to our media pipeline. It's better we take SDP into our sight when we are trying to redesign the whole media pipeline.


##How to Implement a SDP?

https://lkml.org/lkml/2015/5/5/551


##SDP on AOSP

On Android, ION can be treated as a implementation of secure data path: https://wiki.linaro.org/BenjaminGaignard/ion

* ION is the memory manager of Android, it could be used by graphic and multimedia stacks to allocate buffers.
* ION include a buffer sharing mechanism between process and drivers.
* ION define opaque handles to manage underline buffers.
* ION handles are only map in kernel if that is needed by drivers, it help to save logical address space.

From the description, it seems that ION can provide a isolated "heap" to the SDP users.
