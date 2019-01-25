# ContentProvider 启动过程





**frameworks/base/core/Java/android/content/ContextWrapper.java**

```java
    @Override
    public ContentResolver getContentResolver() {
        return mBase.getContentResolver();
    }
```



**frameworks/base/core/java/android/app/ContextImpl.java**

```java
  @Override
    public ContentResolver getContentResolver() {
        return mContentResolver;
    }
```



**frameworks/base/core/java/android/content/ContentResolver.java**

```java
public final @Nullable Cursor query(final @RequiresPermission.Read @NonNull Uri uri,
            @Nullable String[] projection, @Nullable Bundle queryArgs,
            @Nullable CancellationSignal cancellationSignal) {
        Preconditions.checkNotNull(uri, "uri");
        IContentProvider unstableProvider = acquireUnstableProvider(uri);
        if (unstableProvider == null) {
            return null;
        }
        IContentProvider stableProvider = null;
        Cursor qCursor = null;
        try {
            long startTime = SystemClock.uptimeMillis();

            ICancellationSignal remoteCancellationSignal = null;
            if (cancellationSignal != null) {
                cancellationSignal.throwIfCanceled();
                remoteCancellationSignal = unstableProvider.createCancellationSignal();
                cancellationSignal.setRemote(remoteCancellationSignal);
            }
            try {
                qCursor = unstableProvider.query(mPackageName, uri, projection,
                        queryArgs, remoteCancellationSignal);
            } catch (DeadObjectException e) {
                // The remote process has died...  but we only hold an unstable
                // reference though, so we might recover!!!  Let's try!!!!
                // This is exciting!!1!!1!!!!1
                unstableProviderDied(unstableProvider);
                stableProvider = acquireProvider(uri);
                if (stableProvider == null) {
                    return null;
                }
                qCursor = stableProvider.query(
                        mPackageName, uri, projection, queryArgs, remoteCancellationSignal);
            }
            if (qCursor == null) {
                return null;
            }

            // Force query execution.  Might fail and throw a runtime exception here.
            qCursor.getCount();
            long durationMillis = SystemClock.uptimeMillis() - startTime;
            maybeLogQueryToEventLog(durationMillis, uri, projection, queryArgs);

            // Wrap the cursor object into CursorWrapperInner object.
            final IContentProvider provider = (stableProvider != null) ? stableProvider
                    : acquireProvider(uri);
            final CursorWrapperInner wrapper = new CursorWrapperInner(qCursor, provider);
            stableProvider = null;
            qCursor = null;
            return wrapper;
        } catch (RemoteException e) {
            // Arbitrary and not worth documenting, as Activity
            // Manager will kill this process shortly anyway.
            return null;
        } finally {
            if (qCursor != null) {
                qCursor.close();
            }
            if (cancellationSignal != null) {
                cancellationSignal.setRemote(null);
            }
            if (unstableProvider != null) {
                releaseUnstableProvider(unstableProvider);
            }
            if (stableProvider != null) {
                releaseProvider(stableProvider);
            }
        }
    }
```





```java
  public final IContentProvider acquireUnstableProvider(Uri uri) {
        if (!SCHEME_CONTENT.equals(uri.getScheme())) {
            return null;
        }
        String auth = uri.getAuthority();
        if (auth != null) {
            return acquireUnstableProvider(mContext, uri.getAuthority());
        }
        return null;
    }
```



**frameworks/base/core/java/android/app/ContextImpl.java**

```java
 @Override
        protected IContentProvider acquireUnstableProvider(Context c, String auth) {
            return mMainThread.acquireProvider(c,
                    ContentProvider.getAuthorityWithoutUserId(auth),
                    resolveUserIdFromAuthority(auth), false);
        }

```



**frameworks/base/core/java/android/app/ActivityThread.java**

```java
 public final IContentProvider acquireProvider(
            Context c, String auth, int userId, boolean stable) {
        final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);
        if (provider != null) {
            return provider;
        }

        // There is a possible race here.  Another thread may try to acquire
        // the same provider at the same time.  When this happens, we want to ensure
        // that the first one wins.
        // Note that we cannot hold the lock while acquiring and installing the
        // provider since it might take a long time to run and it could also potentially
        // be re-entrant in the case where the provider is in the same process.
        ContentProviderHolder holder = null;
        try {
            holder = ActivityManager.getService().getContentProvider(
                    getApplicationThread(), auth, userId, stable);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
        if (holder == null) {
            Slog.e(TAG, "Failed to find provider info for " + auth);
            return null;
        }

        // Install provider will increment the reference count for us, and break
        // any ties in the race.
        holder = installProvider(c, holder, holder.info,
                true /*noisy*/, holder.noReleaseNeeded, stable);
        return holder.provider;
    }
```



**frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java**

```java
@Override
    public final ContentProviderHolder getContentProvider(
            IApplicationThread caller, String name, int userId, boolean stable) {
        enforceNotIsolatedCaller("getContentProvider");
        if (caller == null) {
            String msg = "null IApplicationThread when getting content provider "
                    + name;
            Slog.w(TAG, msg);
            throw new SecurityException(msg);
        }
        // The incoming user check is now handled in checkContentProviderPermissionLocked() to deal
        // with cross-user grant.
        return getContentProviderImpl(caller, name, null, stable, userId);
    }
```





```java

    private ContentProviderHolder getContentProviderImpl(IApplicationThread caller,
            String name, IBinder token, boolean stable, int userId) {
        ContentProviderRecord cpr;
        ContentProviderConnection conn = null;
        ProviderInfo cpi = null;

        synchronized(this) {
            long startTime = SystemClock.uptimeMillis();

            ProcessRecord r = null;
            if (caller != null) {
                r = getRecordForAppLocked(caller);
                if (r == null) {
                    throw new SecurityException(
                            "Unable to find app for caller " + caller
                          + " (pid=" + Binder.getCallingPid()
                          + ") when getting content provider " + name);
                }
            }

            boolean checkCrossUser = true;

            checkTime(startTime, "getContentProviderImpl: getProviderByName");

            // First check if this content provider has been published...
            cpr = mProviderMap.getProviderByName(name, userId);
            // If that didn't work, check if it exists for user 0 and then
            // verify that it's a singleton provider before using it.
            if (cpr == null && userId != UserHandle.USER_SYSTEM) {
                cpr = mProviderMap.getProviderByName(name, UserHandle.USER_SYSTEM);
                if (cpr != null) {
                    cpi = cpr.info;
                    if (isSingleton(cpi.processName, cpi.applicationInfo,
                            cpi.name, cpi.flags)
                            && isValidSingletonCall(r.uid, cpi.applicationInfo.uid)) {
                        userId = UserHandle.USER_SYSTEM;
                        checkCrossUser = false;
                    } else {
                        cpr = null;
                        cpi = null;
                    }
                }
            }

            boolean providerRunning = cpr != null && cpr.proc != null && !cpr.proc.killed;
            if (providerRunning) {
                cpi = cpr.info;
                String msg;
                checkTime(startTime, "getContentProviderImpl: before checkContentProviderPermission");
                if ((msg = checkContentProviderPermissionLocked(cpi, r, userId, checkCrossUser))
                        != null) {
                    throw new SecurityException(msg);
                }
                checkTime(startTime, "getContentProviderImpl: after checkContentProviderPermission");

                if (r != null && cpr.canRunHere(r)) {
                    // This provider has been published or is in the process
                    // of being published...  but it is also allowed to run
                    // in the caller's process, so don't make a connection
                    // and just let the caller instantiate its own instance.
                    ContentProviderHolder holder = cpr.newHolder(null);
                    // don't give caller the provider object, it needs
                    // to make its own.
                    holder.provider = null;
                    return holder;
                }
                // Don't expose providers between normal apps and instant apps
                try {
                    if (AppGlobals.getPackageManager()
                            .resolveContentProvider(name, 0 /*flags*/, userId) == null) {
                        return null;
                    }
                } catch (RemoteException e) {
                }

                final long origId = Binder.clearCallingIdentity();

                checkTime(startTime, "getContentProviderImpl: incProviderCountLocked");

                // In this case the provider instance already exists, so we can
                // return it right away.
                conn = incProviderCountLocked(r, cpr, token, stable);
                if (conn != null && (conn.stableCount+conn.unstableCount) == 1) {
                    if (cpr.proc != null && r.setAdj <= ProcessList.PERCEPTIBLE_APP_ADJ) {
                        // If this is a perceptible app accessing the provider,
                        // make sure to count it as being accessed and thus
                        // back up on the LRU list.  This is good because
                        // content providers are often expensive to start.
                        checkTime(startTime, "getContentProviderImpl: before updateLruProcess");
                        updateLruProcessLocked(cpr.proc, false, null);
                        checkTime(startTime, "getContentProviderImpl: after updateLruProcess");
                    }
                }

                checkTime(startTime, "getContentProviderImpl: before updateOomAdj");
                final int verifiedAdj = cpr.proc.verifiedAdj;
                boolean success = updateOomAdjLocked(cpr.proc, true);
                // XXX things have changed so updateOomAdjLocked doesn't actually tell us
                // if the process has been successfully adjusted.  So to reduce races with
                // it, we will check whether the process still exists.  Note that this doesn't
                // completely get rid of races with LMK killing the process, but should make
                // them much smaller.
                if (success && verifiedAdj != cpr.proc.setAdj && !isProcessAliveLocked(cpr.proc)) {
                    success = false;
                }
                maybeUpdateProviderUsageStatsLocked(r, cpr.info.packageName, name);
                checkTime(startTime, "getContentProviderImpl: after updateOomAdj");
                if (DEBUG_PROVIDER) Slog.i(TAG_PROVIDER, "Adjust success: " + success);
                // NOTE: there is still a race here where a signal could be
                // pending on the process even though we managed to update its
                // adj level.  Not sure what to do about this, but at least
                // the race is now smaller.
                if (!success) {
                    // Uh oh...  it looks like the provider's process
                    // has been killed on us.  We need to wait for a new
                    // process to be started, and make sure its death
                    // doesn't kill our process.
                    Slog.i(TAG, "Existing provider " + cpr.name.flattenToShortString()
                            + " is crashing; detaching " + r);
                    boolean lastRef = decProviderCountLocked(conn, cpr, token, stable);
                    checkTime(startTime, "getContentProviderImpl: before appDied");
                    appDiedLocked(cpr.proc);
                    checkTime(startTime, "getContentProviderImpl: after appDied");
                    if (!lastRef) {
                        // This wasn't the last ref our process had on
                        // the provider...  we have now been killed, bail.
                        return null;
                    }
                    providerRunning = false;
                    conn = null;
                } else {
                    cpr.proc.verifiedAdj = cpr.proc.setAdj;
                }

                Binder.restoreCallingIdentity(origId);
            }

            if (!providerRunning) {
                try {
                    checkTime(startTime, "getContentProviderImpl: before resolveContentProvider");
                    cpi = AppGlobals.getPackageManager().
                        resolveContentProvider(name,
                            STOCK_PM_FLAGS | PackageManager.GET_URI_PERMISSION_PATTERNS, userId);
                    checkTime(startTime, "getContentProviderImpl: after resolveContentProvider");
                } catch (RemoteException ex) {
                }
                if (cpi == null) {
                    return null;
                }
                // If the provider is a singleton AND
                // (it's a call within the same user || the provider is a
                // privileged app)
                // Then allow connecting to the singleton provider
                boolean singleton = isSingleton(cpi.processName, cpi.applicationInfo,
                        cpi.name, cpi.flags)
                        && isValidSingletonCall(r.uid, cpi.applicationInfo.uid);
                if (singleton) {
                    userId = UserHandle.USER_SYSTEM;
                }
                cpi.applicationInfo = getAppInfoForUser(cpi.applicationInfo, userId);
                checkTime(startTime, "getContentProviderImpl: got app info for user");

                String msg;
                checkTime(startTime, "getContentProviderImpl: before checkContentProviderPermission");
                if ((msg = checkContentProviderPermissionLocked(cpi, r, userId, !singleton))
                        != null) {
                    throw new SecurityException(msg);
                }
                checkTime(startTime, "getContentProviderImpl: after checkContentProviderPermission");

                if (!mProcessesReady
                        && !cpi.processName.equals("system")) {
                    // If this content provider does not run in the system
                    // process, and the system is not yet ready to run other
                    // processes, then fail fast instead of hanging.
                    throw new IllegalArgumentException(
                            "Attempt to launch content provider before system ready");
                }

                // Make sure that the user who owns this provider is running.  If not,
                // we don't want to allow it to run.
                if (!mUserController.isUserRunningLocked(userId, 0)) {
                    Slog.w(TAG, "Unable to launch app "
                            + cpi.applicationInfo.packageName + "/"
                            + cpi.applicationInfo.uid + " for provider "
                            + name + ": user " + userId + " is stopped");
                    return null;
                }

                ComponentName comp = new ComponentName(cpi.packageName, cpi.name);
                checkTime(startTime, "getContentProviderImpl: before getProviderByClass");
                cpr = mProviderMap.getProviderByClass(comp, userId);
                checkTime(startTime, "getContentProviderImpl: after getProviderByClass");
                final boolean firstClass = cpr == null;
                if (firstClass) {
                    final long ident = Binder.clearCallingIdentity();

                    // If permissions need a review before any of the app components can run,
                    // we return no provider and launch a review activity if the calling app
                    // is in the foreground.
                    if (mPermissionReviewRequired) {
                        if (!requestTargetProviderPermissionsReviewIfNeededLocked(cpi, r, userId)) {
                            return null;
                        }
                    }

                    try {
                        checkTime(startTime, "getContentProviderImpl: before getApplicationInfo");
                        ApplicationInfo ai =
                            AppGlobals.getPackageManager().
                                getApplicationInfo(
                                        cpi.applicationInfo.packageName,
                                        STOCK_PM_FLAGS, userId);
                        checkTime(startTime, "getContentProviderImpl: after getApplicationInfo");
                        if (ai == null) {
                            Slog.w(TAG, "No package info for content provider "
                                    + cpi.name);
                            return null;
                        }
                        ai = getAppInfoForUser(ai, userId);
                        cpr = new ContentProviderRecord(this, cpi, ai, comp, singleton);
                    } catch (RemoteException ex) {
                        // pm is in same process, this will never happen.
                    } finally {
                        Binder.restoreCallingIdentity(ident);
                    }
                }

                checkTime(startTime, "getContentProviderImpl: now have ContentProviderRecord");

                if (r != null && cpr.canRunHere(r)) {
                    // If this is a multiprocess provider, then just return its
                    // info and allow the caller to instantiate it.  Only do
                    // this if the provider is the same user as the caller's
                    // process, or can run as root (so can be in any process).
                    return cpr.newHolder(null);
                }

                if (DEBUG_PROVIDER) Slog.w(TAG_PROVIDER, "LAUNCHING REMOTE PROVIDER (myuid "
                            + (r != null ? r.uid : null) + " pruid " + cpr.appInfo.uid + "): "
                            + cpr.info.name + " callers=" + Debug.getCallers(6));

                // This is single process, and our app is now connecting to it.
                // See if we are already in the process of launching this
                // provider.
                final int N = mLaunchingProviders.size();
                int i;
                for (i = 0; i < N; i++) {
                    if (mLaunchingProviders.get(i) == cpr) {
                        break;
                    }
                }

                // If the provider is not already being launched, then get it
                // started.
                if (i >= N) {
                    final long origId = Binder.clearCallingIdentity();

                    try {
                        // Content provider is now in use, its package can't be stopped.
                        try {
                            checkTime(startTime, "getContentProviderImpl: before set stopped state");
                            AppGlobals.getPackageManager().setPackageStoppedState(
                                    cpr.appInfo.packageName, false, userId);
                            checkTime(startTime, "getContentProviderImpl: after set stopped state");
                        } catch (RemoteException e) {
                        } catch (IllegalArgumentException e) {
                            Slog.w(TAG, "Failed trying to unstop package "
                                    + cpr.appInfo.packageName + ": " + e);
                        }

                        // Use existing process if already started
                        checkTime(startTime, "getContentProviderImpl: looking for process record");
                        ProcessRecord proc = getProcessRecordLocked(
                                cpi.processName, cpr.appInfo.uid, false);
                        if (proc != null && proc.thread != null && !proc.killed) {
                            if (DEBUG_PROVIDER) Slog.d(TAG_PROVIDER,
                                    "Installing in existing process " + proc);
                            if (!proc.pubProviders.containsKey(cpi.name)) {
                                checkTime(startTime, "getContentProviderImpl: scheduling install");
                                proc.pubProviders.put(cpi.name, cpr);
                                try {
                                    proc.thread.scheduleInstallProvider(cpi);
                                } catch (RemoteException e) {
                                }
                            }
                        } else {
                            checkTime(startTime, "getContentProviderImpl: before start process");
                            proc = startProcessLocked(cpi.processName,
                                    cpr.appInfo, false, 0, "content provider",
                                    new ComponentName(cpi.applicationInfo.packageName,
                                            cpi.name), false, false, false);
                            checkTime(startTime, "getContentProviderImpl: after start process");
                            if (proc == null) {
                                Slog.w(TAG, "Unable to launch app "
                                        + cpi.applicationInfo.packageName + "/"
                                        + cpi.applicationInfo.uid + " for provider "
                                        + name + ": process is bad");
                                return null;
                            }
                        }
                        cpr.launchingApp = proc;
                        mLaunchingProviders.add(cpr);
                    } finally {
                        Binder.restoreCallingIdentity(origId);
                    }
                }

                checkTime(startTime, "getContentProviderImpl: updating data structures");

                // Make sure the provider is published (the same provider class
                // may be published under multiple names).
                if (firstClass) {
                    mProviderMap.putProviderByClass(comp, cpr);
                }

                mProviderMap.putProviderByName(name, cpr);
                conn = incProviderCountLocked(r, cpr, token, stable);
                if (conn != null) {
                    conn.waiting = true;
                }
            }
            checkTime(startTime, "getContentProviderImpl: done!");

            grantEphemeralAccessLocked(userId, null /*intent*/,
                    cpi.applicationInfo.uid, UserHandle.getAppId(Binder.getCallingUid()));
        }

        // Wait for the provider to be published...
        synchronized (cpr) {
            while (cpr.provider == null) {
                if (cpr.launchingApp == null) {
                    Slog.w(TAG, "Unable to launch app "
                            + cpi.applicationInfo.packageName + "/"
                            + cpi.applicationInfo.uid + " for provider "
                            + name + ": launching app became null");
                    EventLog.writeEvent(EventLogTags.AM_PROVIDER_LOST_PROCESS,
                            UserHandle.getUserId(cpi.applicationInfo.uid),
                            cpi.applicationInfo.packageName,
                            cpi.applicationInfo.uid, name);
                    return null;
                }
                try {
                    if (DEBUG_MU) Slog.v(TAG_MU,
                            "Waiting to start provider " + cpr
                            + " launchingApp=" + cpr.launchingApp);
                    if (conn != null) {
                        conn.waiting = true;
                    }
                    cpr.wait();
                } catch (InterruptedException ex) {
                } finally {
                    if (conn != null) {
                        conn.waiting = false;
                    }
                }
            }
        }
        return cpr != null ? cpr.newHolder(conn) : null;
    }
```



```java
final ProcessRecord getProcessRecordLocked(String processName, int uid, boolean keepIfLarge) {
        if (uid == SYSTEM_UID) {
            // The system gets to run in any process.  If there are multiple
            // processes with the same uid, just pick the first (this
            // should never happen).
            SparseArray<ProcessRecord> procs = mProcessNames.getMap().get(processName);
            if (procs == null) return null;
            final int procCount = procs.size();
            for (int i = 0; i < procCount; i++) {
                final int procUid = procs.keyAt(i);
                if (UserHandle.isApp(procUid) || !UserHandle.isSameUser(procUid, uid)) {
                    // Don't use an app process or different user process for system component.
                    continue;
                }
                return procs.valueAt(i);
            }
        }
        ProcessRecord proc = mProcessNames.get(processName, uid);
        if (false && proc != null && !keepIfLarge
                && proc.setProcState >= ActivityManager.PROCESS_STATE_CACHED_EMPTY
                && proc.lastCachedPss >= 4000) {
            // Turn this condition on to cause killing to happen regularly, for testing.
            if (proc.baseProcessTracker != null) {
                proc.baseProcessTracker.reportCachedKill(proc.pkgList, proc.lastCachedPss);
            }
            proc.kill(Long.toString(proc.lastCachedPss) + "k from cached", true);
        } else if (proc != null && !keepIfLarge
                && mLastMemoryLevel > ProcessStats.ADJ_MEM_FACTOR_NORMAL
                && proc.setProcState >= ActivityManager.PROCESS_STATE_CACHED_EMPTY) {
            if (DEBUG_PSS) Slog.d(TAG_PSS, "May not keep " + proc + ": pss=" + proc.lastCachedPss);
            if (proc.lastCachedPss >= mProcessList.getCachedRestoreThresholdKb()) {
                if (proc.baseProcessTracker != null) {
                    proc.baseProcessTracker.reportCachedKill(proc.pkgList, proc.lastCachedPss);
                }
                proc.kill(Long.toString(proc.lastCachedPss) + "k from cached", true);
            }
        }
        return proc;
    }
```

