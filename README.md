# > Local vagrant environment `IDM` SIT merge branch run

## Setup:

</br>

`ORACLE`

```
$ docker image ls
> vfzccam.azurecr.io/ci-support/oracle                  12c-20230213-1-pre                6dd63e12e90c   2 months ago    8.27GB
```

- The `12c-20230213-1-pre` image digest id matches the one the pipeline has cached.

```
$ docker image ls
> registry.gitlab.com/vodafoneziggodi/ciam/support-containers/ci-support/oracle            12c-20230213-1-pre                          6dd63e12e90c        2 months ago        8.27GB
```

---

`CIAM:`

```
Run locally in host mode on `merge/sit/23-q2` branch (equivilant to image tag when ran in pipeline: 5.9.10-sit-23-q2.8)
```

---

`IDM:`

```
Run locally in host mode on `merge/sit/23-q2` branch (same branch we are trying to get working in pipeline)
```

- `clean build` results:

```
367 tests completed, 3 failed, 1 skipped
```

- The failed tests in this case are large tests that time out / fail due to my local machine being overwhelmed running `ciam` and `idm` locally at the same time, they fail unpredictably sometimes as well.

```
> Task :idm:integrationTest

nl.vodafoneziggo.ponoi.api.exposed.mass_invite.service.MassInviteSummaryTest > thatMassInviteSummaryFeedbackIsSentCorrectly FAILED
    java.lang.AssertionError at MassInviteSummaryTest.java:70

nl.vodafoneziggo.ponoi.api.exposed.mass_invite.service.MassInviteSummaryTest > thatMassInviteSummaryFeedbackIsSendCorrectlyWhenExceedeWindow FAILED
    java.lang.AssertionError at MassInviteSummaryTest.java:103

nl.vodafoneziggo.ponoi.api.exposed.sso.new_device_login.NewDeviceLoginEventProducerTest > An event is sent when a logs onto a new device FAILED
    org.spockframework.runtime.ConditionFailedWithExceptionError at NewDeviceLoginEventProducerTest.groovy:38
        Caused by: java.util.concurrent.TimeoutException at NewDeviceLoginEventProducerTest.groovy:38

```

</br>

## Conclusion:

### IDM `clean build` result: `success`

`(couple flakey integration test errors due to overwhelmed local machine)`

```
367 tests completed, 3 failed, 1 skipped
```

</br>
</br>

## This suggests that every dependancy that is needed for the `IDM` SIT merge branch to run successfully is playing nice with each other and should work...

</br>
</br>

---

</br>
</br>

# > Gitlab `IDM` pipeline run `master` branch

</br>

## `master_new_oracle` branch result: `failed`

(`master` branch run result : `success`)

</br>

## Setup difference compared to `master` branch:

</br>

`ORACLE`

```
  services:
    - name: $CI_REGISTRY/vodafoneziggodi/ciam/support-containers/ci-support/oracle:12c-20230213-1-pre
      alias: oracle-db
```

- difference is that the `oracle` tag is '`master`' in the `master` branch

</br>

## Pipeline

example: `https://gitlab.com/vodafoneziggodi/ciam/pono-i/-/pipelines/864369822`

```
180 failed integration tests
```

- The only change is the `oracle` image tag which is used in the local vagrant environment with an IDENTICAL image digest id `6dd63e12e90c` and this results in a consistent pipeline fail due to integration test fails.

</br>
</br>

---

</br>
</br>

# > Next steps / tasks

[] - Confirm that `oracle` container is started successfully during pipeline run with tag `12c-20230213-1-pre`.

[] - If the container is started successfully then why are other services/containers not able to communicate with it.

[] - Confirm that connection to `oracle` is made successfully by `CIAM` container when starting.

[] - Once `oracle:12c-20230213-1-pre` works in `master_new_oracle` we can then try bumping `ciam` image tag to `5.9.10-sit-23-q2.8` and also merging in `add_redis_secondary` in since redis secondary is a dependency for the new `ciam` tag and seeing if `ciam` can connect to the new redis instance.

[] - Finally we can then merge all these changes into a new `merge/23-q2_test` branch where we have all the depenencies that are working prior and we can test if `IDM` `merge/sit/23-q2` works 😄
