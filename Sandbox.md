##Overview with Sandbox
For Gecko, each platform has its own Sandbox mechanism:
* **B2G/Gonk** : gecko derived the android id (AID) mechanism, the child process is forked with different AID to be sandboxed. As we know, the predefined AID is related to different selinux policies.
* **Linux** : gecko use seccomp to make sandbox work.
* **Mac** : gecko use seatbelt to make sandbox work.

##What's the differences for GMP child process?
* On Mac & Linux, GMP child process can only access to the plugin file (shared library), child has no privilege to load other libraries dynamically (by libdl), but Gonk has no such limitation, maybe selinux policy is not that strong as Mac & Linux.

##How to disable Sandbox in a GMP plugin
* **B2G/Gonk** : disable SetCurrentProcessPrivileges() defined in ipc/chromium/src/base/process_util_linux.cc
* **Linux&Mac** : disable SandboxStarter when child process is forked (ipc/contentproc/plugin-container.cpp)
<pre>
@@ -231,7 +238,7 @@ content_process_main(int argc, char* argv[])
     // code can be covered by an EME/GMP vendor's voucher.
     nsAutoPtr<mozilla::gmp::SandboxStarter> starter(MakeSandboxStarter());
     if (XRE_GetProcessType() == GeckoProcessType_GMPlugin) {
-        loader = mozilla::gmp::CreateGMPLoader(starter);
+        loader = mozilla::gmp::CreateGMPLoader(nullptr);
     }
 #endif
</pre>

##A intro to Seccomp##
A wiki page form mozilla:
https://wiki.mozilla.org/Security/Sandbox/Seccomp

##How does Sandbox work##
**H5OS's Sandbox**
* SetPrivilege() right after ChildProcess is forked
* MakeSandboxStarter() and pass it to the ChildLoader

Different platform have their own Sandbox implementation.
* On linux, they are using Seccomp as a kernel enhancement.
* On Mac, os x has its own Sandbox framework

reference:
* https://www.chromium.org/developers/design-documents/sandbox/osx-sandboxing-design
* https://reverse.put.as/wp-content/uploads/2011/06/The-Apple-Sandbox-BHDC2011-Paper.pdf

##Sandbox on Mac

As for sandbox on Mac OS X, something should be noted:
* Sandboxing an application begins with a call to the system function sandbox_init (libsandbox.dylib).
* A policy definition should be passed to sandbox_init describing rules like “don’t allow access to files under /opt/sekret”
* The policy definition is written in scheme, and libsandbox.dylib will turn it into a binary format used by kernel
* sandbox-exec provides three ways to specify the access policy:
** by naming a built-in profile (such as “no-internet” or “pure-computation”)
** by providing the path to a configuration file
** by giving the configuration directly as a string.

Lets take a look with sandbox rules in H5OS:
* **plugin child sandbox rules**
<pre>
(version 1)
(deny default)
(allow signal (target self))
(allow sysctl-read)
%s(allow iokit-open (iokit-user-client-class "IOHIDParamUserClient"))
%s(allow file-read-data (literal "%s"))
(allow mach-lookup
    (global-name "com.apple.cfprefsd.agent")
    (global-name "com.apple.cfprefsd.daemon")
    (global-name "com.apple.system.opendirectoryd.libinfo")
    (global-name "com.apple.system.logger")
    (global-name "com.apple.ls.boxd"))
(allow file-read*
    (regex #"^/etc$")
    (regex #"^/dev/u?random$")
    (regex #"^/(private/)?var($|/)")
    (literal "/usr/share/icu/icudt51l.dat")
    (regex #"^/System/Library/Displays/Overrides/*")
    (regex #"^/System/Library/CoreServices/CoreTypes.bundle/*")
    (literal "%s")
    (literal "%s")
    (literal "%s"))
</pre>


* **content child sandbox rules**
<pre>
(version 1)

(define sandbox-level %d)
(define macosMinorVersion %d)
(define appPath "%s")
(define appBinaryPath "%s")
(define appDir "%s")
(define home-path "%s")

(import "/System/Library/Sandbox/Profiles/system.sb")

(if 
  (or
    (< macosMinorVersion 9)
    (< sandbox-level 1))
  (allow default)
  (begin
    (deny default)
    (debug deny)

    (define resolving-literal literal)
    (define resolving-subpath subpath)
    (define resolving-regex regex)

    (define container-path appPath)
    (define appdir-path appDir)
    (define var-folders-re "^/private/var/folders/[^/][^/]")
    (define var-folders2-re (string-append var-folders-re "/[^/]+/[^/]"))

    (define (home-regex home-relative-regex)
      (resolving-regex (string-append "^" (regex-quote home-path) home-relative-regex)))
    (define (home-subpath home-relative-subpath)
      (resolving-subpath (string-append home-path home-relative-subpath)))
    (define (home-literal home-relative-literal)
      (resolving-literal (string-append home-path home-relative-literal)))

    (define (container-regex container-relative-regex)
      (resolving-regex (string-append "^" (regex-quote container-path) container-relative-regex)))
    (define (container-subpath container-relative-subpath)
      (resolving-subpath (string-append container-path container-relative-subpath)))
    (define (container-literal container-relative-literal)
      (resolving-literal (string-append container-path container-relative-literal)))

    (define (var-folders-regex var-folders-relative-regex)
      (resolving-regex (string-append var-folders-re var-folders-relative-regex)))
    (define (var-folders2-regex var-folders2-relative-regex)
      (resolving-regex (string-append var-folders2-re var-folders2-relative-regex)))

    (define (appdir-regex appdir-relative-regex)
      (resolving-regex (string-append "^" (regex-quote appdir-path) appdir-relative-regex)))
    (define (appdir-subpath appdir-relative-subpath)
      (resolving-subpath (string-append appdir-path appdir-relative-subpath)))
    (define (appdir-literal appdir-relative-literal)
      (resolving-literal (string-append appdir-path appdir-relative-literal)))

    (define (allow-shared-preferences-read domain)
          (begin
            (if (defined? `user-preference-read)
              (allow user-preference-read (preference-domain domain)))
            (allow file-read*
                   (home-literal (string-append "/Library/Preferences/" domain ".plist"))
                   (home-regex (string-append "/Library/Preferences/ByHost/" (regex-quote domain) "\..*\.plist$")))
            ))

    (define (allow-shared-list domain)
      (allow file-read*
             (home-regex (string-append "/Library/Preferences/" (regex-quote domain)))))

    (allow file-read-metadata)

    (allow ipc-posix-shm
        (ipc-posix-name-regex "^/tmp/com.apple.csseed:")
        (ipc-posix-name-regex "^CFPBS:")
        (ipc-posix-name-regex "^AudioIO"))

    (allow file-read-metadata
        (literal "/home")
        (literal "/net")
        (regex "^/private/tmp/KSInstallAction\.")
        (var-folders-regex "/")
        (home-subpath "/Library"))

    (allow signal (target self))
    (allow job-creation (literal "/Library/CoreMediaIO/Plug-Ins/DAL"))
    (allow iokit-set-properties (iokit-property "IOAudioControlValue"))

    (allow mach-lookup
        (global-name "com.apple.coreservices.launchservicesd")
        (global-name "com.apple.coreservices.appleevents")
        (global-name "com.apple.pasteboard.1")
        (global-name "com.apple.window_proxies")
        (global-name "com.apple.windowserver.active")
        (global-name "com.apple.audio.coreaudiod")
        (global-name "com.apple.audio.audiohald")
        (global-name "com.apple.PowerManagement.control")
        (global-name "com.apple.cmio.VDCAssistant")
        (global-name "com.apple.SystemConfiguration.configd")
        (global-name "com.apple.iconservices")
        (global-name "com.apple.cookied")
        (global-name "com.apple.printuitool.agent")
        (global-name "com.apple.printtool.agent")
        (global-name "com.apple.cache_delete")
        (global-name "com.apple.pluginkit.pkd")
        (global-name "com.apple.bird")
        (global-name "com.apple.ocspd")
        (global-name "com.apple.cmio.AppleCameraAssistant")
        (global-name "com.apple.DesktopServicesHelper")
        (global-name "com.apple.printtool.daemon"))

    (allow iokit-open
        (iokit-user-client-class "IOHIDParamUserClient")
        (iokit-user-client-class "IOAudioControlUserClient")
        (iokit-user-client-class "IOAudioEngineUserClient")
        (iokit-user-client-class "IGAccelDevice")
        (iokit-user-client-class "nvDevice")
        (iokit-user-client-class "nvSharedUserClient")
        (iokit-user-client-class "nvFermiGLContext")
        (iokit-user-client-class "IGAccelGLContext")
        (iokit-user-client-class "IGAccelSharedUserClient")
        (iokit-user-client-class "IGAccelVideoContextMain")
        (iokit-user-client-class "IGAccelVideoContextMedia")
        (iokit-user-client-class "IGAccelVideoContextVEBox")
        (iokit-user-client-class "RootDomainUserClient")
        (iokit-user-client-class "IOUSBDeviceUserClientV2")
        (iokit-user-client-class "IOUSBInterfaceUserClientV2"))

; depending on systems, the 1st, 2nd or both rules are necessary
    (allow-shared-preferences-read "com.apple.HIToolbox")
    (allow file-read-data (literal "/Library/Preferences/com.apple.HIToolbox.plist"))

    (allow-shared-preferences-read "com.apple.ATS")
    (allow file-read-data (literal "/Library/Preferences/.GlobalPreferences.plist"))

    (allow file-read*
        (subpath "/Library/Fonts")
        (subpath "/Library/Audio/Plug-Ins")
        (subpath "/Library/CoreMediaIO/Plug-Ins/DAL")
        (subpath "/Library/Spelling")
        (subpath "/private/etc/cups/ppd")
        (subpath "/private/var/run/cupsd")
        (literal "/")
        (literal "/private/tmp")
        (literal "/private/var/tmp")

        (home-literal "/.CFUserTextEncoding")
        (home-literal "/Library/Preferences/com.apple.DownloadAssessment.plist")
        (home-subpath "/Library/Colors")
        (home-subpath "/Library/Fonts")
        (home-subpath "/Library/FontCollections")
        (home-subpath "/Library/Keyboard Layouts")
        (home-subpath "/Library/Input Methods")
        (home-subpath "/Library/PDF Services")
        (home-subpath "/Library/Spelling")

        (subpath appdir-path)

        (literal appPath)
        (literal appBinaryPath))

    (allow-shared-list "org.mozilla.plugincontainer")

; the following 2 rules should be removed when microphone and camera access
; are brokered through the content process
    (allow device-microphone)
    (allow device-camera)

    (allow file* (var-folders2-regex "/com\.apple\.IntlDataCache\.le$"))
    (allow file-read*
        (var-folders2-regex "/com\.apple\.IconServices/")
        (var-folders2-regex "/[^/]+\.mozrunner/extensions/[^/]+/chrome/[^/]+/content/[^/]+\.j(s|ar)$"))

    (allow file-write* (var-folders2-regex "/org\.chromium\.[a-zA-Z0-9]*$"))
    (allow file-read*
        (home-regex "/Library/Application Support/[^/]+/Extensions/[^/]/")
        (resolving-regex "/Library/Application Support/[^/]+/Extensions/[^/]/")
        (home-regex "/Library/Application Support/Firefox/Profiles/[^/]+/extensions/")
        (home-regex "/Library/Application Support/Firefox/Profiles/[^/]+/weave/"))

; the following rules should be removed when printing and 
; opening a file from disk are brokered through the main process
    (if
      (< sandbox-level 2)
      (allow file*
          (require-not
              (home-subpath "/Library")))
      (allow file*
          (require-all
              (subpath home-path)
              (require-not
                  (home-subpath "/Library")))))

; printing
    (allow authorization-right-obtain
           (right-name "system.print.operator")
           (right-name "system.printingmanager"))
    (allow mach-lookup
           (global-name "com.apple.printuitool.agent")
           (global-name "com.apple.printtool.agent")
           (global-name "com.apple.printtool.daemon")
           (global-name "com.apple.sharingd")
           (global-name "com.apple.metadata.mds")
           (global-name "com.apple.mtmd.xpc")
           (global-name "com.apple.FSEvents")
           (global-name "com.apple.locum")
           (global-name "com.apple.ImageCaptureExtension2.presence"))
    (allow file-read*
           (home-literal "/.cups/lpoptions")
           (home-literal "/.cups/client.conf")
           (literal "/private/etc/cups/lpoptions")
           (literal "/private/etc/cups/client.conf")
           (subpath "/private/etc/cups/ppd")
           (literal "/private/var/run/cupsd"))
    (allow-shared-preferences-read "org.cups.PrintingPrefs")
    (allow-shared-preferences-read "com.apple.finder")
    (allow-shared-preferences-read "com.apple.LaunchServices")
    (allow-shared-preferences-read ".GlobalPreferences")
    (allow network-outbound
        (literal "/private/var/run/cupsd")
        (literal "/private/var/run/mDNSResponder"))

; print preview
    (if (> macosMinorVersion 9)
        (allow lsopen))
    (allow file-write* file-issue-extension (var-folders2-regex "/"))
    (allow file-read-xattr (literal "/Applications/Preview.app"))
    (allow mach-task-name)
    (allow mach-register)
    (allow file-read-data
        (regex "^/Library/Printers/[^/]+/PDEs/[^/]+.plugin")
        (subpath "/Library/PDF Services")
        (subpath "/Applications/Preview.app")
        (home-literal "/Library/Preferences/com.apple.ServicesMenu.Services.plist"))
    (allow mach-lookup
        (global-name "com.apple.pbs.fetch_services")
        (global-name "com.apple.tsm.uiserver")
        (global-name "com.apple.ls.boxd")
        (global-name "com.apple.coreservices.quarantine-resolver")
        (global-name-regex "_OpenStep$"))
    (allow appleevent-send
        (appleevent-destination "com.apple.preview")
        (appleevent-destination "com.apple.imagecaptureextension2"))

; accelerated graphics
    (allow-shared-preferences-read "com.apple.opengl")
    (allow-shared-preferences-read "com.nvidia.OpenGL")
    (allow mach-lookup
        (global-name "com.apple.cvmsServ"))
    (allow iokit-open
        (iokit-connection "IOAccelerator")
        (iokit-user-client-class "IOAccelerationUserClient")
        (iokit-user-client-class "IOSurfaceRootUserClient")
        (iokit-user-client-class "IOSurfaceSendRight")
        (iokit-user-client-class "IOFramebufferSharedUserClient")
        (iokit-user-client-class "AppleSNBFBUserClient")
        (iokit-user-client-class "AGPMClient")
        (iokit-user-client-class "AppleGraphicsControlClient")
        (iokit-user-client-class "AppleGraphicsPolicyClient"))

; bug 1153809
    (allow iokit-open
        (iokit-user-client-class "NVDVDContextTesla")
        (iokit-user-client-class "Gen6DVDContext"))

; bug 1190032
    (allow file*
        (home-regex "/Library/Caches/TemporaryItems/plugtmp.*"))

; bug 1201935
    (allow file-read*
        (home-subpath "/Library/Caches/TemporaryItems"))
  )
)
</pre>

