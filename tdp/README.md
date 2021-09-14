# Knox TDP

## Build

```
mvn clean install \
  -Prelease \
  -Ppackage \
  -Drat.numUnapprovedLicenses=1000 \
  -DskipTests \
  -Dmaven.javdoc.skip=true \
  -Dcheckstyle.skip=true \
  -Dfindbugs.skip=true \
  -Dspotbugs.skip=true \
  --batch-mode \
  -fae
```

## Test

```
mvn clean test \
  -Panalyze \
  --batch-mode \
  --fail-never
```

## Skipped tests

* `org.apache.hadoop.gateway.shirorealm.KnoxPamRealmTest.testDoGetAuthenticationInfo`

```java
  @Test
  public void testDoGetAuthenticationInfo() {
    KnoxPamRealm realm = new KnoxPamRealm();
    realm.setService("sshd");  // pam settings being used: /etc/pam.d/sshd

    // use environment variables and skip the test if not set.
    String pamuser = System.getenv("PAMUSER");
    String pampass = System.getenv("PAMPASS");
    assumeTrue(pamuser != null);
    assumeTrue(pampass != null);
```
