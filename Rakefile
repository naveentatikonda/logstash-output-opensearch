# SPDX-License-Identifier: Apache-2.0
#
#  The OpenSearch Contributors require contributions made to
#  this file be licensed under the Apache-2.0 license or a
#  compatible open source license.
#
#  Modifications Copyright OpenSearch Contributors. See
#  GitHub history for details.

require "logstash/devutils/rake"

ECS_VERSIONS = {
    v1: 'v1.5.0'
}

ECS_LOGSTASH_INDEX_PATTERNS = %w(
  ecs-logstash-*
)

task :'vendor-ecs-schemata' do
  download_ecs_schema(:v1)
end
task :vendor => :'vendor-ecs-schemata'

def download_ecs_schema(ecs_major_version)
  $stderr.puts("Vendoring ECS #{ecs_major_version} template")
  require 'net/http'
  require 'json'
  Net::HTTP.start('raw.githubusercontent.com', :use_ssl => true) do |http|
    ecs_release_tag = ECS_VERSIONS.fetch(ecs_major_version)
    response = http.get("/elastic/ecs/#{ecs_release_tag}/generated/elasticsearch/7/template.json")
    fail "#{response.code} #{response.message}" unless (200...300).cover?(response.code.to_i)
    template_directory = File.expand_path("../lib/logstash/outputs/opensearch/templates/ecs-#{ecs_major_version}", __FILE__)
    Dir.mkdir(template_directory) unless File.exists?(template_directory)
    File.open(File.join(template_directory, "/template.json"), "w") do |handle|
      handle.write(replace_index_patterns(response.body, ECS_LOGSTASH_INDEX_PATTERNS))
    end
  end
end

def replace_index_patterns(template_json, replacement_index_patterns)
  template_obj = JSON.load(template_json)
  template_obj.update('index_patterns' => replacement_index_patterns)
  JSON.pretty_generate(template_obj)
end
