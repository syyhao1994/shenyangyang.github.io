enum JobType {
        JOB_START,                  /* if a unit does not support being started, we'll just wait until it becomes active */
        JOB_VERIFY_ACTIVE,

​        JOB_STOP,

​        JOB_RELOAD,                 /* if running, reload */

​        /* Note that restarts are first treated like JOB_STOP, but
         * then instead of finishing are patched to become
                  * JOB_START. */
                JOB_RESTART,                /* If running, stop. Then start unconditionally. */

​        _JOB_TYPE_MAX_MERGING,

​        /* JOB_NOP can enter into a transaction, but as it won't pull in
         * any dependencies and it uses the special 'nop_job' slot in Unit,
                  * it won't have to merge with anything (except possibly into another
                  * JOB_NOP, previously installed). JOB_NOP is special-cased in
                           * job_type_is_*() functions so that the transaction can be
                           * activated. */
                                JOB_NOP = _JOB_TYPE_MAX_MERGING, /* do nothing */

​        _JOB_TYPE_MAX_IN_TRANSACTION,

​        /* JOB_TRY_RESTART can never appear in a transaction, because
         * it always collapses into JOB_RESTART or JOB_NOP before entering.
                  * Thus we never need to merge it with anything. */
                JOB_TRY_RESTART = _JOB_TYPE_MAX_IN_TRANSACTION, /* if running, stop and then start */

​        /* Similar to JOB_TRY_RESTART but collapses to JOB_RELOAD or JOB_NOP */
​        JOB_TRY_RELOAD,

​        /* JOB_RELOAD_OR_START won't enter into a transaction and cannot result
         * from transaction merging (there's no way for JOB_RELOAD and
                  * JOB_START to meet in one transaction). It can result from a merge
                  * during job installation, but then it will immediately collapse into
                           * one of the two simpler types. */
                        JOB_RELOAD_OR_START,        /* if running, reload, otherwise start */

​        _JOB_TYPE_MAX,
​        _JOB_TYPE_INVALID = -EINVAL,
};

enum JobState {
        JOB_WAITING,
        JOB_RUNNING,
        _JOB_STATE_MAX,
        _JOB_STATE_INVALID = -EINVAL,
};

enum JobMode {
        JOB_FAIL,                /* Fail if a conflicting job is already queued */
        JOB_REPLACE,             /* Replace an existing conflicting job */
        JOB_REPLACE_IRREVERSIBLY,/* Like JOB_REPLACE + produce irreversible jobs */
        JOB_ISOLATE,             /* Start a unit, and stop all others */
        JOB_FLUSH,               /* Flush out all other queued jobs when queueing this one */
        JOB_IGNORE_DEPENDENCIES, /* Ignore both requirement and ordering dependencies */
        JOB_IGNORE_REQUIREMENTS, /* Ignore requirement dependencies */
        JOB_TRIGGERING,          /* Adds TRIGGERED_BY dependencies to the same transaction */
        _JOB_MODE_MAX,
        _JOB_MODE_INVALID = -EINVAL,
};

enum JobResult {
        JOB_DONE,                /* Job completed successfully (or skipped due to a failed ConditionXYZ=) */
        JOB_CANCELED,            /* Job canceled by a conflicting job installation or by explicit cancel request */
        JOB_TIMEOUT,             /* Job timeout elapsed */
        JOB_FAILED,              /* Job failed */
        JOB_DEPENDENCY,          /* A required dependency job did not result in JOB_DONE */
        JOB_SKIPPED,             /* Negative result of JOB_VERIFY_ACTIVE or skip due to ExecCondition= */
        JOB_INVALID,             /* JOB_RELOAD of inactive unit */
        JOB_ASSERT,              /* Couldn't start a unit, because an assert didn't hold */
        JOB_UNSUPPORTED,         /* Couldn't start a unit, because the unit type is not supported on the system */
        JOB_COLLECTED,           /* Job was garbage collected, since nothing needed it anymore */
        JOB_ONCE,                /* Unit was started before, and hence can't be started again */
        _JOB_RESULT_MAX,
        _JOB_RESULT_INVALID = -EINVAL,
};

#include "unit.h"

struct JobDependency {
        /* Encodes that the 'subject' job needs the 'object' job in
         * some way. This structure is used only while building a transaction. */
                Job *subject;
                Job *object;

​        LIST_FIELDS(JobDependency, subject);
​        LIST_FIELDS(JobDependency, object);

​        bool matters:1;
​        bool conflicts:1;
};

struct Job {
        Manager *manager;
        Unit *unit;

​        LIST_FIELDS(Job, transaction);
​        LIST_FIELDS(Job, dbus_queue);
​        LIST_FIELDS(Job, gc_queue);

​        LIST_HEAD(JobDependency, subject_list);
​        LIST_HEAD(JobDependency, object_list);

​        /* Used for graph algs as a "I have been here" marker */
​        Job* marker;
​        unsigned generation;

​        uint32_t id;

​        JobType type;
​        JobState state;

​        sd_event_source *timer_event_source;
​        usec_t begin_usec;
​        usec_t begin_running_usec;

​        /*
         * This tracks where to send signals, and also which clients
                  * are allowed to call DBus methods on the job (other than
                  * root).
                           *
                           * There can be more than one client, because of job merging.
                                    */
                        ​        sd_bus_track *bus_track;
                        ​        char **deserialized_clients;

​        JobResult result;

​        unsigned run_queue_idx;

​        bool installed:1;
​        bool in_run_queue:1;
​        bool matters_to_anchor:1;
​        bool in_dbus_queue:1;
​        bool sent_dbus_new_signal:1;
​        bool ignore_order:1;
​        bool irreversible:1;
​        bool in_gc_queue:1;
​        bool ref_by_private_bus:1;
};