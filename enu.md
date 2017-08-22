# ENU SRS format specification

ENU SRS is a local cartesian (XYZ) system around a point of origin placed on a reference spheroid (ellipsoid). By default the WGS84 sheroid is used.

ENU SRS string is defined as follows:

 * starts with `enu`
 * contains a reference spheroid (defaults to WGS84), either none or both `a` and `b` must be used:
     * `a=A` semimajor axis of the reference spheroid
     * `b=B` semiminor axis of the reference spheroid
 * contains an origin specification
     * `lon0=x` specifies longitude of origin on the reference sheroid (`lon0` defaults to 0)
     * `lat0=y` specifies latitude of origin on the reference sheroid (`lat0` defaults to 0)
     * `h0=z` specifies height of origin above the reference sheroid (`h0` defaults to 0)
 * contains optional `towgs84=TOWGS84` definition, see Proj.4 documentation for details (comma-separated list of either 3 or 7 real numbers)

XYZ coordinate system for given ENU SRS definition has the origin at (lon0, lat0, h0) point on the reference spheroid. XY plane is tangent to the spheroid at (lon0, lat0) coordinates and Z-axis is perpendicular to this plane.

Geographiclib's [GeographicLib::LocalCartesian](http://geographiclib.sourceforge.net/html/classGeographicLib_1_1LocalCartesian.html) class can be used for convesion between latlon and enu coordinates (in both directions): `lat0`, `lon0`, `h0` map to constructor parameters and `a` and `b` can be used to construct [GeographicLib::GeoCentric](http://geographiclib.sourceforge.net/html/classGeographicLib_1_1Geocentric.html) instance (`f = (a - b) / a`), use `GeographicLib::Geocentric::WGS84` for default spheroid.

When converting coordinates between ENU SRS and another SRS one must convert coordinates between ENU SRS and intermediate latlon SRS and then between intermediate latlon SRS and destination SRS. Nota bene that intermediate latlon SRS is defined as:
```
+proj=lonlat +a=A +b=B +towgs84=TOGWS84
```
where spheroid definition (+a, +b) and towgs84 transformation are optional (based on ENU SRS).
