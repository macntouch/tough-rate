Ruleset Loader
==============

    find_rule_in = require '../find_rule_in'

    class CCNQBaseMiddlewareError extends Error

    module.exports = (provisioning,ruleset_of,default_outbound_route) ->
      assert provisioning, 'Missing provisioning'
      assert ruleset_of, 'Missing ruleset_of'

We first need to determine which routing table we should use, though.
This is based on the calling number.

      middleware = ->

        return if @finalized()

Route based on the route selected by the source, or using a default route.

        source = @source

        provisioning.get "number:#{source}"
        .then (doc) =>
          route = doc.outbound_route ? default_outbound_route
        .catch =>
          route = default_outbound_route
        .then (route) =>
          unless route?
            @respond '485'
            @logger.warn 'No route available', {source}
            throw new CCNQBaseMiddlewareError "No route available for #{source}"

          route = "#{route}"
          @res.route = route

          ruleset_of route
        .then ({ruleset,ruleset_database}) =>
          unless ruleset? and ruleset_database?
            @respond '500'
            @logger.warn 'No ruleset available', {source,route:@res.route,ruleset,ruleset_database}
            throw new CCNQBaseMiddlewareError "Route `#{@res.route}` for `#{source}` has no ruleset or no database."

          @res.ruleset = ruleset
          @res.ruleset_database = ruleset_database

          find_rule_in @res.destination,ruleset_database
        .then (rule) =>
          unless rule?
            @respond '485'
            @logger.warn 'No route available', {source,destination:@res.destination,ruleset:@res.ruleset}
            throw new CCNQBaseMiddlewareError "No rule available towards #{@res.destination}"

          if rule.gwlist?
            @res.gateways = rule.gwlist
            delete rule.gwlist
          else
            @logger.warn 'Missing gwlist', rule

          @res.rule = rule

        .catch (error) =>
          @logger.error "Ruleset middleware failed:", error


    assert = require 'assert'
    field_merger = require '../field_merger'