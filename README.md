# GeoPostGIS

[![Build Status](https://travis-ci.org/bryanjos/geo_postgis.svg?branch=master)](https://travis-ci.org/bryanjos/geo_postgis)

Postgrex extension for the PostGIS data types. Uses the [geo](https://github.com/bryanjos/geo) library

[Documentation](http://hexdocs.pm/geo_postgis)

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `geo_postgis` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:geo_postgis, "~> 1.0"}]
end
```

## Examples

### Postgrex Extension for the PostGIS data types, Geometry and Geography

  ```elixir
  Postgrex.Types.define(MyApp.PostgresTypes, [Geo.PostGIS.Extension], [])

  opts = [hostname: "localhost", username: "postgres", database: "geo_postgrex_test", types: MyApp.PostgresTypes ]
  [hostname: "localhost", username: "postgres", database: "geo_postgrex_test", types: MyApp.PostgresTypes]

  {:ok, pid} = Postgrex.Connection.start_link(opts)
  {:ok, #PID<0.115.0>}

  geo = %Geo.Point{coordinates: {30, -90}, srid: 4326}
  %Geo.Point{coordinates: {30, -90}, srid: 4326}

  {:ok, _} = Postgrex.Connection.query(pid, "CREATE TABLE point_test (id int, geom geometry(Point, 4326))")
  {:ok, %Postgrex.Result{columns: nil, command: :create_table, num_rows: 0, rows: nil}}

  {:ok, _} = Postgrex.Connection.query(pid, "INSERT INTO point_test VALUES ($1, $2)", [42, geo])
  {:ok, %Postgrex.Result{columns: nil, command: :insert, num_rows: 1, rows: nil}}

  Postgrex.Connection.query(pid, "SELECT * FROM point_test")
  {:ok, %Postgrex.Result{columns: ["id", "geom"], command: :select, num_rows: 1,
  rows: [{42, %Geo.Point{coordinates: {30.0, -90.0}, srid: 4326 }}]}}
  ```

### Use with Ecto Referencing [the documentation](https://hexdocs.pm/ecto/Ecto.Adapters.Postgres.html#module-extensions):

  ```elixir

  #If using with Ecto, you may want something like thing instead
  Postgrex.Types.define(MyApp.PostgresTypes,
                [Geo.PostGIS.Extension] ++ Ecto.Adapters.Postgres.extensions(),
                json: Poison)

  #Add extensions to your repo config
  config :thanks, Repo,
    database: "geo_postgrex_test",
    username: "postgres",
    password: "postgres",
    hostname: "localhost",
    adapter: Ecto.Adapters.Postgres,
    types: MyApp.PostgresTypes


  #Create a model
  defmodule Test do
    use Ecto.Model

    schema "test" do
      field :name,           :string
      field :geom,           Geo.Geometry
    end
  end

  #Geometry or Geography columns can be created in migrations too
  defmodule Repo.Migrations.Init do
    use Ecto.Migration

    def up do
      create table(:test) do
        add :name,     :string
        add :geom,     :geometry
      end
    end

    def down do
      drop table(:test)
    end
  end
  ```


### Ecto migrations can also use more elaborate [Postgis GIS Objects](http://postgis.net/docs/using_postgis_dbmanagement.html#RefObject). These types are useful for enforcing constraints on {Lng,Lat} (order matters), or ensuring that a particular projection/coordinate system/format is used.

  ```elixir
  defmodule Repo.Migrations.AdvancedInit do
    use Ecto.Migration

    def up do
      create table(:test) do
        add :name,     :string
      end
      # Add a field `lng_lat_point` with type `geometry(Point,4326)`.
      # This can store a "standard GPS" (epsg4326) coordinate pair {longitude,latitude}.
      execute("SELECT AddGeometryColumn ('test','lng_lat_point',4326,'POINT',2);")
    end

    def down do
      drop table(:test)
    end
  end
  ```

  Be sure to enable the Postgis extension if you haven't already done so:

  ```elixir

  defmodule MyApp.Repo.Migrations.EnablePostgis do
    use Ecto.Migration

    def up do
      execute "CREATE EXTENSION IF NOT EXISTS postgis"
    end

    def down do
      execute "DROP EXTENSION IF EXISTS postgis"
    end
  end
  ```

### [Postgis functions](http://postgis.net/docs/manual-1.3/ch06.html) can also be used in ecto queries. Currently only the OpenGIS functions are implemented. Have a look at [lib/geo/postgis.ex](lib/geo/postgis.ex) for the implemented functions. You can use them like:

  ```elixir
  defmodule Example do
    import Ecto.Query
    import Geo.PostGIS

    def example_query(geom) do
      from location in Location, limit: 5, select: st_distance(location.geom, ^geom)
    end

  end
  ```
